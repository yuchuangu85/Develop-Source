<h1 align="center">深入理解Binder通信原理及面试问题</h1>

[toc]

Binder承担了绝大部分Android进程通信的职责，可以看做是Android的血管系统，负责不同服务模块进程间的通信。在对Binder的理解上，可大可小，日常APP开发并不怎么涉及Binder通信知识，最多就是Service及AIDL的使用会涉及部分Binder知识。Binder往小了说可总结成一句话：一种IPC进程间通信方式，负责进程A的数据，发送到进程B。往大了说，其实涉及的知识还是很多的，如Android 对于原Binder驱动的扩展、Zygote进程孵化中对于Binder通信的支持、Java层Binder封装，Native层对于Binder通信的封装、Binder讣告机制等等。很多分析Binder框架的文都是从ServiceManager、Binder驱动、addService、getService来分析等来分析，其实这些主要是针对系统提供的服务，但是bindService启动的服务走的却还是有很大不同的。本篇文章主要简述一些Binder难以理解的点，但不会太细的跟踪分析，只抛砖，自己去发掘玉：

- **Binder的定向制导，如何找到目标Binder，唤起进程或者线程**
- **Binder中的红黑树，为什么会有两棵binder_ref红黑树**
- **Binder一次拷贝原理(直接拷贝到目标线程的内核空间，内核空间与用户空间对应)**
- **Binder传输数据的大小限制（内核4M 上层限制1m-8k），传输Bitmap过大，就会崩溃的原因，Activity之间传输BitMap**
- **系统服务与bindService等启动的服务的区别**
- **Binder线程、Binder主线程、Client请求线程的概念与区别**
- **Client是同步而Server是异步到底说的什么**
- **Android APP进程天生支持Binder通信的原理是什么**
- **Android APP有多少Binder线程，是固定的么**
- **Binder线程的睡眠与唤醒（请求线程睡在哪个等待队列上，唤醒目标端哪个队列上的线程）**
- **Binder协议中BC与BR的区别**
- **Binder在传输数据的时候是如何层层封装的–不同层次使用的数据结构（命令的封装**）
- **Binder驱动传递数据的释放（释放时机）**
- **一个简单的Binder通信C/S模型**
- **ServiceManager addService的限制（并非服务都能使用ServiceManager的addService）**
- **bindService启动Service与Binder服务实体的流程**
- **Java层Binder实体与与BinderProxy是如何实例化及使用的，与Native层的关系是怎样的**
- **Parcel readStrongBinder与writeStrongBinder的原理**

## Binder如何精确制导，找到目标Binder实体，并唤醒进程或者线程

Binder实体服务其实有两种，一是通过addService注册到ServiceManager中的服务，比如ActivityManagerService、PackageManagerService、PowerManagerService等，一般都是系统服务；还有一种是通过bindService拉起的一些服务，一般是开发者自己实现的服务。这里先看通过addService添加的被ServiceManager所管理的服务。有很多分析ServiceManager的文章，本文不分析ServiceManager，只是简单提一下，ServiceManager是比较特殊的服务，所有应用都能直接使用，因为ServiceManager对于Client端来说Handle句柄是固定的，都是0，所以ServiceManager服务并不需要查询，可以直接使用。

理解Binder定向制导的关键是理解Binder的四棵红黑树，先看一下binder_proc结构体，在它内部有四棵红黑树，threads，nodes，refs_by_desc，refs_by_node，nodes就是Binder实体在内核中对应的数据结构，binder_node里记录进程相关的binder_proc，还有Binder实体自身的地址等信息，nodes红黑树位于binder_proc，可以知道Binder实体其实是进程内可见，而不是线程内。

```c++
struct binder_proc {
	struct hlist_node proc_node;
	struct rb_root threads;
	struct rb_root nodes;
	struct rb_root refs_by_desc;
	struct rb_root refs_by_node;
	。。。
	struct list_head todo;
	wait_queue_head_t wait;
	。。。
};
```

现在假设存在一堆Client与Service，Client如何才能访问Service呢？首先Service会通过addService将binder实体注册到ServiceManager中去，Client如果想要使用Servcie，就需要通过getService向ServiceManager请求该服务。在Service通过addService向ServiceManager注册的时候，ServiceManager会将服务相关的信息存储到自己进程的Service列表中去，同时在ServiceManager进程的binder_ref红黑树中为Service添加binder_ref节点，这样ServiceManager就能获取Service的Binder实体信息。而当Client通过getService向ServiceManager请求该Service服务的时候，ServiceManager会在注册的Service列表中查找该服务，如果找到就将该服务返回给Client，在这个过程中，ServiceManager会在Client进程的binder_ref红黑树中添加binder_ref节点，可见**本进程中的binder_ref红黑树节点都不是本进程自己创建的，要么是Service进程将binder_ref插入到ServiceManager中去，要么是ServiceManager进程将binder_ref插入到Client中去**。之后，Client就能通过Handle句柄获取binder_ref，进而访问Service服务。

<img src="media/format,png.jpeg" alt="binder_ref添加逻辑" style="zoom:50%;" />

getService之后，便可以获取binder_ref引用，进而获取到binder_proc与binder_node信息，之后Client便可有目的的将binder_transaction事务插入到binder_proc的待处理列表，并且，如果进程正在睡眠，就唤起进程，其实这里到底是唤起进程还是线程也有讲究，对于Client向Service发送请求的状况，一般都是唤醒binder_proc上睡眠的线程：

```c++
struct binder_ref {
	int debug_id;
	struct rb_node rb_node_desc;
	struct rb_node rb_node_node;
	struct hlist_node node_entry;
	struct binder_proc *proc;
	struct binder_node *node;
	uint32_t desc;
	int strong;
	int weak;
	struct binder_ref_death *death;
};
```

## binder_proc为何会有两棵binder_ref红黑树

binder_proc中存在两棵binder_ref红黑树，其实两棵红黑树中的节点是复用的，只是查询方式不同，一个通过handle句柄，一个通过node节点查找。个人理解：refs_by_node红黑树主要是为了
binder驱动往用户空间写数据所使用的，而refs_by_desc是用户空间向Binder驱动写数据使用的，只是方向问题。比如在服务addService的时候，binder驱动会在在ServiceManager进程的binder_proc中查找binder_ref结构体，如果没有就会新建binder_ref结构体，再比如在Client端getService的时候，binder驱动会在Client进程中通过 binder_get_ref_for_node为Client创建binder_ref结构体，并分配句柄，同时插入到refs_by_desc红黑树中，可见refs_by_node红黑树，主要是给binder驱动往用户空间写数据使用的。相对的refs_by_desc主要是为了用户空间往binder驱动写数据使用的，当用户空间已经获得Binder驱动为其创建的binder_ref引用句柄后，就可以通过binder_get_ref从refs_by_desc找到响应binder_ref，进而找到目标binder_node。可见有两棵红黑树主要是区分使用对象及数据流动方向，看下面的代码就能理解：

```c++
// 根据32位的uint32_t desc来查找，可以看到，binder_get_ref不会新建binder_ref节点
static struct binder_ref *binder_get_ref(struct binder_proc *proc,
					 uint32_t desc)
{
	struct rb_node *n = proc->refs_by_desc.rb_node;
	struct binder_ref *ref;
	while (n) {
		ref = rb_entry(n, struct binder_ref, rb_node_desc);
		if (desc < ref->desc)
			n = n->rb_left;
		else if (desc > ref->desc)
			n = n->rb_right;
		else
			return ref;
	}
	return NULL;
}
```

可以看到binder_get_ref并具备binder_ref的创建功能，相对应的看一下binder_get_ref_for_node，binder_get_ref_for_node红黑树主要通过binder_node进行查找，如果找不到，就新建binder_ref，同时插入到两棵红黑树中去

```c++
static struct binder_ref *binder_get_ref_for_node(struct binder_proc *proc,
						  struct binder_node *node)
{
	struct rb_node *n;
	struct rb_node **p = &proc->refs_by_node.rb_node;
	struct rb_node *parent = NULL;
	struct binder_ref *ref, *new_ref;
	while (*p) {
		parent = *p;
		ref = rb_entry(parent, struct binder_ref, rb_node_node);
		if (node < ref->node)
			p = &(*p)->rb_left;
		else if (node > ref->node)
			p = &(*p)->rb_right;
		else
			return ref;
	}

	// binder_ref 可以在两棵树里面，但是，两棵树的查询方式不同，并且通过desc查询，不具备新建功能
	new_ref = kzalloc(sizeof(*ref), GFP_KERNEL);
	if (new_ref == NULL)
		return NULL;
	binder_stats_created(BINDER_STAT_REF);
	new_ref->debug_id = ++binder_last_id;
	new_ref->proc = proc;
	new_ref->node = node;
	rb_link_node(&new_ref->rb_node_node, parent, p);
	// 插入到proc->refs_by_node红黑树中去
	rb_insert_color(&new_ref->rb_node_node, &proc->refs_by_node);
	// 是不是ServiceManager的
	new_ref->desc = (node == binder_context_mgr_node) ? 0 : 1;
	// 分配Handle句柄，为了插入到refs_by_desc
	for (n = rb_first(&proc->refs_by_desc); n != NULL; n = rb_next(n)) {
		ref = rb_entry(n, struct binder_ref, rb_node_desc);
		if (ref->desc > new_ref->desc)
			break;
		new_ref->desc = ref->desc + 1;
	}
	// 找到目标位置
	p = &proc->refs_by_desc.rb_node;
	while (*p) {
		parent = *p;
		ref = rb_entry(parent, struct binder_ref, rb_node_desc);
		if (new_ref->desc < ref->desc)
			p = &(*p)->rb_left;
		else if (new_ref->desc > ref->desc)
			p = &(*p)->rb_right;
		else
			BUG();
	}
	rb_link_node(&new_ref->rb_node_desc, parent, p);
	// 插入到refs_by_desc红黑树中区
	rb_insert_color(&new_ref->rb_node_desc, &proc->refs_by_desc);

	if (node) {
		hlist_add_head(&new_ref->node_entry, &node->refs);
		binder_debug(BINDER_DEBUG_INTERNAL_REFS,
			     "binder: %d new ref %d desc %d for "
			     "node %d\n", proc->pid, new_ref->debug_id,
			     new_ref->desc, node->debug_id);
	} else {
		binder_debug(BINDER_DEBUG_INTERNAL_REFS,
			     "binder: %d new ref %d desc %d for "
			     "dead node\n", proc->pid, new_ref->debug_id,
			      new_ref->desc);
	}
	return new_ref;
}
```

该函数调用在binder_transaction函数中，其实就是在binder驱动访问target_proc的时候，这也也很容易理解，Handle句柄对于跨进程没有任何意义，进程A中的Handle，放到进程B中是无效的。

<img src="media/format,png-20201231114140007.jpeg" alt="两棵binder_ref红黑树" style="zoom:67%;" />

## Binder一次拷贝原理

Android选择Binder作为主要进程通信的方式同其性能高也有关系，Binder只需要一次拷贝就能将A进程用户空间的数据为B进程所用。这里主要涉及两个点：

- Binder的map函数，会将内核空间直接与用户空间对应，用户空间可以直接访问内核空间的数据

- A进程的数据会被直接拷贝到B进程的内核空间（一次拷贝）

   ```c++
     #define BINDER_VM_SIZE ((1*1024*1024) - (4096 *2))
     
     ProcessState::ProcessState()
         : mDriverFD(open_driver())
         , mVMStart(MAP_FAILED)
         , mManagesContexts(false)
         , mBinderContextCheckFunc(NULL)
         , mBinderContextUserData(NULL)
         , mThreadPoolStarted(false)
         , mThreadPoolSeq(1){
        if (mDriverFD >= 0) {
     	....
             // mmap the binder, providing a chunk of virtual address space to receive transactions.
             mVMStart = mmap(0, BINDER_VM_SIZE, PROT_READ, MAP_PRIVATE | MAP_NORESERVE, mDriverFD, 0);
        ...
      
         }
     }
   ```

mmap函数属于系统调用，mmap会从当前进程中获取用户态可用的虚拟地址空间（vm_area_struct *vma），并在mmap_region中真正获取vma，然后调用file->f_op->mmap(file, vma)，进入驱动处理，之后就会在内存中分配一块连续的虚拟地址空间，并预先分配好页表、已使用的与未使用的标识、初始地址、与用户空间的偏移等等，通过这一步之后，就能把Binder在内核空间的数据直接通过指针地址映射到用户空间，供进程在用户空间使用，这是一次拷贝的基础，一次拷贝在内核中的标识如下：

```c++
	struct binder_proc {
	struct hlist_node proc_node;
	// 四棵比较重要的树 
	struct rb_root threads;
	struct rb_root nodes;
	struct rb_root refs_by_desc;
	struct rb_root refs_by_node;
	int pid;
	struct vm_area_struct *vma; //虚拟地址空间，用户控件传过来
	struct mm_struct *vma_vm_mm;
	struct task_struct *tsk;
	struct files_struct *files;
	struct hlist_node deferred_work_node;
	int deferred_work;
	void *buffer; //初始地址
	ptrdiff_t user_buffer_offset; //这里是偏移
	
	struct list_head buffers;//这个列表连接所有的内存块，以地址的大小为顺序，各内存块首尾相连
	struct rb_root free_buffers;//连接所有的已建立映射的虚拟内存块，以内存的大小为index组织在以该节点为根的红黑树下
	struct rb_root allocated_buffers;//连接所有已经分配的虚拟内存块，以内存块的开始地址为index组织在以该节点为根的红黑树下
	
	}
```

上面只是在APP启动的时候开启的地址映射，但并未涉及到数据的拷贝，下面看数据的拷贝操作。**当数据从用户空间拷贝到内核空间的时候，是直从当前进程的用户空间接拷贝到目标进程的内核空间，这个过程是在请求端线程中处理的，操作对象是目标进程的内核空间**。看如下代码：

```c++
static void binder_transaction(struct binder_proc *proc,
			       struct binder_thread *thread,
			       struct binder_transaction_data *tr, int reply){
			       ...
		在通过进行binder事物的传递时，如果一个binder事物（用struct binder_transaction结构体表示）需要使用到内存，
		就会调用binder_alloc_buf函数分配此次binder事物需要的内存空间。
		需要注意的是：这里是从目标进程的binder内存空间分配所需的内存
		//从target进程的binder内存空间分配所需的内存大小,这也是一次拷贝，完成通信的关键，直接拷贝到目标进程的内核空间
		//由于用户空间跟内核空间仅仅存在一个偏移地址，所以也算拷贝到用户空间
		t->buffer = binder_alloc_buf(target_proc, tr->data_size,
			tr->offsets_size, !reply && (t->flags & TF_ONE_WAY));
		t->buffer->allow_user_free = 0;
		t->buffer->debug_id = t->debug_id;
		//该binder_buffer对应的事务    
		t->buffer->transaction = t;
		//该事物对应的目标binder实体 ,因为目标进程中可能不仅仅有一个Binder实体
		t->buffer->target_node = target_node;
		trace_binder_transaction_alloc_buf(t->buffer);
		if (target_node)
			binder_inc_node(target_node, 1, 0, NULL);
		// 计算出存放flat_binder_object结构体偏移数组的起始地址，4字节对齐。
		offp = (size_t *)(t->buffer->data + ALIGN(tr->data_size, sizeof(void *)));
		   // struct flat_binder_object是binder在进程之间传输的表示方式 //
	       // 这里就是完成binder通讯单边时候在用户进程同内核buffer之间的一次拷贝动作 //
		  // 这里的数据拷贝，其实是拷贝到目标进程中去，因为t本身就是在目标进程的内核空间中分配的，
		if (copy_from_user(t->buffer->data, tr->data.ptr.buffer, tr->data_size)) {
			binder_user_error("binder: %d:%d got transaction with invalid "
				"data ptr\n", proc->pid, thread->pid);
			return_error = BR_FAILED_REPLY;
			goto err_copy_data_failed;
		}
```

可以看到binder_alloc_buf(target_proc, tr->data_size,tr->offsets_size, !reply && (t->flags & TF_ONE_WAY))函数在申请内存的时候，是从target_proc进程空间中去申请的，这样在做数据拷贝的时候copy_from_user(t->buffer->data, tr->data.ptr.buffer, tr->data_size))，就会直接拷贝target_proc的内核空间，而由于Binder内核空间的数据能直接映射到用户空间，这里就不在需要拷贝到用户空间。这就是一次拷贝的原理。内核空间的数据映射到用户空间其实就是添加一个偏移地址，并且将数据的首地址、数据的大小都复制到一个用户空间的Parcel结构体，具体可以参考Parcel.cpp的Parcel::ipcSetDataReference函数。

<img src="media/format,png-20201231114140605.jpeg" alt="Binder一次拷贝原理.jpg" style="zoom:50%;" />

## Binder传输数据的大小限制

虽然APP开发时候，Binder对程序员几乎不可见，但是作为Android的数据运输系统，Binder的影响是全面性的，所以有时候如果不了解Binder的一些限制，在出现问题的时候往往是没有任何头绪，比如在Activity之间传输BitMap的时候，如果Bitmap过大，就会引起问题，比如崩溃等，这其实就跟Binder传输数据大小的限制有关系，在上面的一次拷贝中分析过，mmap函数会为Binder数据传递映射一块连续的虚拟地址，这块虚拟内存空间其实是有大小限制的，不同的进程可能还不一样。

普通的由Zygote孵化而来的用户进程，所映射的Binder内存大小是不到1M的，准确说是 1*1024*1024) - (4096 *2) ：这个限制定义在ProcessState类中，如果传输说句超过这个大小，系统就会报错，因为Binder本身就是为了进程间频繁而灵活的通信所设计的，并不是为了拷贝大数据而使用的：

```c++
#define BINDER_VM_SIZE ((1*1024*1024) - (4096 *2))
```

而在内核中，其实也有个限制，是4M，不过由于APP中已经限制了不到1M，这里的限制似乎也没多大用途：

```c++
static int binder_mmap(struct file *filp, struct vm_area_struct *vma)
{
	int ret;
	struct vm_struct *area;
	struct binder_proc *proc = filp->private_data;
	const char *failure_string;
	struct binder_buffer *buffer;
    //限制不能超过4M
	if ((vma->vm_end - vma->vm_start) > SZ_4M)
		vma->vm_end = vma->vm_start + SZ_4M;
	。。。
}
```

有个特殊的进程ServiceManager进程，它为自己申请的Binder内核空间是128K，这个同ServiceManager的用途是分不开的，ServcieManager主要面向系统Service，只是简单的提供一些addServcie，getService的功能，不涉及多大的数据传输，因此不需要申请多大的内存：

```c++
int main(int argc, char **argv)
{
    struct binder_state *bs;
    void *svcmgr = BINDER_SERVICE_MANAGER;

		// 仅仅申请了128k
    bs = binder_open(128*1024);
    if (binder_become_context_manager(bs)) {
        ALOGE("cannot become context manager (%s)\n", strerror(errno));
        return -1;
    }

    svcmgr_handle = svcmgr;
    binder_loop(bs, svcmgr_handler);
    return 0;
}
```

## 系统服务与bindService等启动的服务的区别

服务可分为系统服务与普通服务，系统服务一般是在系统启动的时候，由SystemServer进程创建并注册到ServiceManager中的。而普通服务一般是通过ActivityManagerService启动的服务，或者说通过四大组件中的Service组件启动的服务。这两种服务在实现跟使用上是有不同的，主要从以下几个方面：

- 服务的启动方式
- 服务的注册与管理
- 服务的请求使用方式

首先看一下服务的启动上，系统服务一般都是SystemServer进程负责启动，比如AMS，WMS，PKMS，电源管理等，这些服务本身其实实现了Binder接口，作为Binder实体注册到ServiceManager中，被ServiceManager管理，而SystemServer进程里面会启动一些Binder线程，主要用于监听Client的请求，并分发给响应的服务实体类，可以看出，这些系统服务是位于SystemServer进程中（有例外，比如Media服务）。在来看一下bindService类型的服务，这类服务一般是通过Activity的startService或者其他context的startService启动的，这里的Service组件只是个封装，主要的是里面Binder服务实体类，这个启动过程不是ServcieManager管理的，而是通过ActivityManagerService进行管理的，同Activity管理类似。

再来看一下服务的注册与管理：系统服务一般都是通过ServiceManager的addService进行注册的，这些服务一般都是需要拥有特定的权限才能注册到ServiceManager，而bindService启动的服务可以算是注册到ActivityManagerService，只不过ActivityManagerService管理服务的方式同ServiceManager不一样，而是采用了Activity的管理模型，详细的可以自行分析

最后看一下使用方式，使用系统服务一般都是通过ServiceManager的getService得到服务的句柄，这个过程其实就是去ServiceManager中查询注册系统服务。而bindService启动的服务，主要是去ActivityManagerService中去查找相应的Service组件，最终会将Service内部Binder的句柄传给Client。

<img src="media/format,png-20201231114140137.jpeg" alt="系统服务与bindService启动服务的区别.jpg" style="zoom: 50%;" />

## Binder线程、Binder主线程、Client请求线程的概念与区别

Binder线程是执行Binder服务的载体，只对于服务端才有意义，对请求端来说，是不需要考虑Binder线程的，但Android系统的处理机制其实大部分是互为C/S的。比如APP与AMS进行交互的时候，都互为对方的C与S，这里先不讨论这个问题，先看Binder线程的概念。

Binder线程就是执行Binder实体业务的线程，一个普通线程如何才能成为Binder线程呢？很简单，只要开启一个监听Binder字符设备的Loop线程即可，在Android中有很多种方法，不过归根到底都是监听Binder，换成代码就是通过ioctl来进行监听。

拿ServerManager进程来说，其主线就是Binder线程，其做法是通过binder_loop实现不死线程：

```c++
void binder_loop(struct binder_state *bs, binder_handler func)
{
   ...
    for (;;) {
    <!--关键点1-->
        res = ioctl(bs->fd, BINDER_WRITE_READ, &bwr);
     <!--关键点2-->
        res = binder_parse(bs, 0, readbuf, bwr.read_consumed, func);
        。。
    }
}
```

上面的关键代码1就是阻塞监听客户端请求，2 就是处理请求，并且这是一个死循环，不退出。再来看SystemServer进程中的线程，在Android4.3（6.0以后打代码就不一样了）中SystemSever主线程便是Binder线程，同时一个Binder主线程，Binder线程与Binder主线程的区别是：线程是否可以终止Loop，不过目前启动的Binder线程都是无法退出的，其实可以全部看做是Binder主线程，其实现原理是，**在SystemServer主线程执行到最后的时候，Loop监听Binder设备，变身死循环线程**，关键代码如下：

```c++
extern "C" status_t system_init()
{
    ...
    ALOGI("System server: entering thread pool.\n");
    ProcessState::self()->startThreadPool();
    IPCThreadState::self()->joinThreadPool();
    ALOGI("System server: exiting thread pool.\n");
    return NO_ERROR;
}
```

ProcessState::self()->startThreadPool()是新建一个Binder主线程，而PCThreadState::self()->joinThreadPool()是将当前线程变成Binder主线程。其实startThreadPool最终也会调用joinThreadPool，看下其关键函数：

```c++
void IPCThreadState::joinThreadPool(bool isMain)
{
 	...
    status_t result;
    do {
        int32_t cmd;
      	...关键点1 
        result = talkWithDriver();
        if (result >= NO_ERROR) {
           ...关键点2 
            result = executeCommand(cmd);
        }
        // 非主线程的可以退出
        if(result == TIMED_OUT && !isMain) {
            break;
        }
        // 死循环，不完结，调用了这个，就好比是开启了Binder监听循环，
    } while (result != -ECONNREFUSED && result != -EBADF);
 }

status_t IPCThreadState::talkWithDriver(bool doReceive)
{  
    do {
        ...关键点3 
        if (ioctl(mProcess->mDriverFD, BINDER_WRITE_READ, &bwr) >= 0)
    }  
}
```

先看关键点1 talkWithDriver，其实质还是去掉用ioctl(mProcess->mDriverFD, BINDER_WRITE_READ, &bwr) >= 0)去不断的监听Binder字符设备，获取到Client传输的数据后，再通过executeCommand去执行相应的请求，joinThreadPool是普通线程化身Binder线程最常见的方式。不信，就再看一个MediaService,看一下main_mediaserver的main函数：

```c++
int main(int argc, char** argv)
{
   ...
    sp<ProcessState> proc(ProcessState::self());
    sp<IServiceManager> sm = defaultServiceManager();
    ALOGI("ServiceManager: %p", sm.get());
    AudioFlinger::instantiate();
    MediaPlayerService::instantiate();
    CameraService::instantiate();
    AudioPolicyService::instantiate();
    registerExtensions();
    ProcessState::self()->startThreadPool();
    IPCThreadState::self()->joinThreadPool();
}
```

其实还是通过joinThreadPool变身Binder线程，至于是不是主线程，看一下下面的函数：

```c++
void IPCThreadState::joinThreadPool(bool isMain)

void ProcessState::spawnPooledThread(bool isMain)
{
    if (mThreadPoolStarted) {
        String8 name = makeBinderThreadName();
        ALOGV("Spawning new pooled thread, name=%s\n", name.string());
        sp<Thread> t = new PoolThread(isMain);
        t->run(name.string());
    }
}
```

其实关键就是就是传递给joinThreadPool函数的isMain是否是true，不过是否是Binder主线程并没有什么用，因为源码中并没有为这两者的不同处理留入口，感兴趣可以去查看一下binder中的TIMED_OUT。

最后来看一下普通Client的binder请求线程，比如我们APP的主线程，在startActivity请求AMS的时候，APP的主线程成其实就是Binder请求线程，在进行Binder通信的过程中，Client的Binder请求线程会一直阻塞，知道Service处理完毕返回处理结果。

## Binder请求的同步与异步

很多人都会说，Binder是对Client端同步，而对Service端异步，其实并不完全正确，在单次Binder数据传递的过程中，其实都是同步的。只不过，Client在请求Server端服务的过程中，是需要返回结果的，即使是你看不到返回数据，其实还是会有个成功与失败的处理结果返回给Client，这就是所说的Client端是同步的。至于说服务端是异步的，可以这么理解：在服务端在被唤醒后，就去处理请求，处理结束后，服务端就将结果返回给正在等待的Client线程，将结果写入到Client的内核空间后，服务端就会直接返回了，不会再等待Client端的确认，这就是所说的服务端是异步的，可以从源码来看一下：

- Client端同步阻塞请求

   ```c++
   status_t IPCThreadState::transact(int32_t handle,
   uint32_t code, const Parcel& data,
   Parcel* reply, uint32_t flags)
   {
   	if (reply) {
             err = waitForResponse(reply);
       }
   } 
   ...
   ```

Client在请求服务的时候 Parcel* reply基本都是非空的（还没见过空用在什么位置），非空就会执行waitForResponse(reply)，如果看过几篇Binder分析文章的人应该都会知道，在A端向B写完数据之后，A会返回给自己一个BR_TRANSACTION_COMPLETE命令，告知自己数据已经成功写入到B的Binder内核空间中去了，如果是需要回复，在处理完BR_TRANSACTION_COMPLETE命令后会继续阻塞等待结果的返回：

```c++
status_t IPCThreadState::waitForResponse(Parcel *reply, status_t *acquireResult){
    ...
    while (1) {
	if ((err=talkWithDriver()) < NO_ERROR) break;
	 cmd = mIn.readInt32();
    switch (cmd) {
	   	 <!--关键点1 -->
         case BR_TRANSACTION_COMPLETE:
            if (!reply && !acquireResult) goto finish;
            break;
         <!--关键点2 -->
         case BR_REPLY:
            {
                binder_transaction_data tr;
                // free buffer，先设置数据，直接
                if (reply) {
                    if ((tr.flags & TF_STATUS_CODE) == 0) {
                        // 牵扯到数据利用，与内存释放
                        reply->ipcSetDataReference(...)
            }
            goto finish;
    }
 	finish:
 	...
	return err;
}
```

关键点1就是处理BR_TRANSACTION_COMPLETE，如果需要等待reply，还要通过talkWithDriver等待结果返回，最后执行关键点2，处理返回数据。**对于服务端来说，区别就在于关键点1 **，来看一下服务端Binder线程的代码，拿常用的joinThreadPool来看，在talkWithDriver后，会执行executeCommand函数，

```c++
void IPCThreadState::joinThreadPool(bool isMain)
{
 	...
    status_t result;
    do {
        int32_t cmd;
      	...关键点1 
        result = talkWithDriver();
        if (result >= NO_ERROR) {
           ...关键点2 
            result = executeCommand(cmd);
        }
        // 非主线程的可以退出
        if(result == TIMED_OUT && !isMain) {
            break;
        }
        // 死循环，不完结，调用了这个，就好比是开启了Binder监听循环，
    } while (result != -ECONNREFUSED && result != -EBADF);
}
```

executeCommand会进一步调用sendReply函数，看一下这里的特点waitForResponse(NULL, NULL)，这里传递的都是null，在上面的关键点1的地方我们知道，这里不需要等待Client返回，因此会直接 goto finish，这就是所说的Client同步，而服务端异步的逻辑。

```c++
// BC_REPLY
status_t IPCThreadState::sendReply(const Parcel& reply, uint32_t flags)
{
    // flag 0
    status_t err;
    status_t statusBuffer;
    err = writeTransactionData(BC_REPLY, flags, -1, 0, reply, &statusBuffer);
    if (err < NO_ERROR) return err;
    return waitForResponse(NULL, NULL);
}
12345678910
  case BR_TRANSACTION_COMPLETE:
        if (!reply && !acquireResult) goto finish;
        break;
```

请求同步最好的例子就是在Android6.0之前，国产ROM权限的申请都是同步的，在申请权限的时候，APP申请权限的线程会阻塞，就算是UI线程也会阻塞，ROM为了防止ANR，都会为权限申请设置一个倒计时，不操作，就给个默认操作，有兴趣可以自己分析。

## Android APP进程天生支持Binder通信的原理是什么

Android APP进程都是由Zygote进程孵化出来的。常见场景：点击桌面icon启动APP，或者startActivity启动一个新进程里面的Activity，最终都会由AMS去调用Process.start()方法去向Zygote进程发送请求，让Zygote去fork一个新进程，Zygote收到请求后会调用Zygote.forkAndSpecialize()来fork出新进程,之后会通过RuntimeInit.nativeZygoteInit来初始化Andriod APP运行需要的一些环境，而binder线程就是在这个时候新建启动的，看下面的源码（Android 4.3）：

这里不分析Zygote，只是给出其大概运行机制，Zygote在启动后，就会通过runSelectLoop不断的监听socket，等待请求来fork进程，如下：

```java
private static void runSelectLoop() throws MethodAndArgsCaller {
    ArrayList<FileDescriptor> fds = new ArrayList<FileDescriptor>();
    ArrayList<ZygoteConnection> peers = new ArrayList<ZygoteConnection>();
    FileDescriptor[] fdArray = new FileDescriptor[4];
	...     
   int loopCount = GC_LOOP_COUNT;
    while (true) {
        int index;
         ...
            boolean done;
            done = peers.get(index).runOnce();
			...
        }
	}
}
```

每次fork请求到来都会调用ZygoteConnection的runOnce()来处理请求，

```java
boolean runOnce() throws ZygoteInit.MethodAndArgsCaller {

    String args[];
    Arguments parsedArgs = null;
    FileDescriptor[] descriptors;	
		。。。

    try {
       ...关键点1 
        pid = Zygote.forkAndSpecialize(parsedArgs.uid, parsedArgs.gid, parsedArgs.gids,
                parsedArgs.debugFlags, rlimits, parsedArgs.mountExternal, parsedArgs.seInfo,
                parsedArgs.niceName);
    } 
    try {
        if (pid == 0) {
            // in child
         ...关键点2
            handleChildProc(parsedArgs, descriptors, childPipeFd, newStderr);
		。。。
    }
```

runOnce()有两个关键点，**关键点1** Zygote.forkAndSpecialize就是通过fork系统调用来新建进程，**关键点2 **handleChildProc就是对新建的APP进程进行一些初始化工作，为Android Java进程创建一些必须的场景。Zygote.forkAndSpecialize没什么可看的，就是Linux中的fork进程，这里主要看一下handleChildProc

```java
private void handleChildProc(Arguments parsedArgs,
        FileDescriptor[] descriptors, FileDescriptor pipeFd, PrintStream newStderr)
        throws ZygoteInit.MethodAndArgsCaller {
        
	//从Process.start启动的parsedArgs.runtimeInit一般都是true    		 if (parsedArgs.runtimeInit) {
        if (parsedArgs.invokeWith != null) {
            WrapperInit.execApplication(parsedArgs.invokeWith,
                    parsedArgs.niceName, parsedArgs.targetSdkVersion,
                    pipeFd, parsedArgs.remainingArgs);  
        } else {
       	 // Android应用启动都走该分支
       RuntimeInit.zygoteInit(parsedArgs.targetSdkVersion,
                    parsedArgs.remainingArgs); 
       }  
}
```

接着看 RuntimeInit.zygoteInit函数

```java
public static final void zygoteInit(int targetSdkVersion, String[] argv)
        throws ZygoteInit.MethodAndArgsCaller {

    redirectLogStreams();
    commonInit();
    <!--关键点1-->
    nativeZygoteInit();
     <!--关键点2-->
    applicationInit(targetSdkVersion, argv);
}
```

先看关键点1，nativeZygoteInit属于Native方法，该方法位于AndroidRuntime.cpp中，其实就是调用调用到app_main.cpp中的onZygoteInit

```c++
static void com_android_internal_os_RuntimeInit_nativeZygoteInit(JNIEnv* env, jobject clazz)
{
    gCurRuntime->onZygoteInit();
}
```

**关键就是onZygoteInit**

```c++
	virtual void onZygoteInit()
	{
	    sp proc = ProcessState::self();
	    //启动新binder线程loop
	    proc->startThreadPool();
	}
```

首先，ProcessState::self()函数会调用open()打开/dev/binder设备，这个时候Client就能通过Binder进行远程通信；其次，proc->startThreadPool()负责新建一个binder线程，监听Binder设备，这样进程就具备了作为Binder服务端的资格。每个APP的进程都会通过onZygoteInit打开Binder，既能作为Client，也能作为Server，这就是Android进程天然支持Binder通信的原因。

![Android APP进程天然支持Binder通信.png](media/format,png.png)

## Android APP有多少Binder线程，是固定的么？

通过上一个问题我们知道了Android APP线程为什么天然支持Binder通信，并且可以作为Binder的Service端，同时也对Binder线程有了一个了解，那么在一个Android APP的进程里面究竟有多少个Binder线程呢？是固定的吗。在分析上一个问题的时候，我们知道Android APP进程在Zygote fork之初就为它新建了一个Binder主线程，使得APP端也可以作为Binder的服务端，这个时候Binder线程的数量就只有一个，假设我们的APP自身实现了很多的Binder服务，一个线程够用的吗？这里不妨想想一下SystemServer进程，SystemServer拥有很多系统服务，一个线程应该是不够用的，如果看过SystemServer代码可能会发现，对于Android4.3的源码，其实一开始为该服务开启了两个Binder线程。还有个分析Binder常用的服务，media服务，也是在一开始的时候开启了两个线程。

先看下SystemServer的开始加载的线程：通过 ProcessState::self()->startThreadPool()新加了一个Binder线程，然后通过IPCThreadState::self()->joinThreadPool();将当前线程变成Binder线程，注意这里是针对Android4.3的源码，android6.0的这里略有不同。

```c++
extern "C" status_t system_init()
{
    ...
    ALOGI("System server: entering thread pool.\n");
    ProcessState::self()->startThreadPool();
    IPCThreadState::self()->joinThreadPool();
    ALOGI("System server: exiting thread pool.\n");
    return NO_ERROR;
}
```

再看下Media服务,同SystemServer类似，也是开启了两个Binder线程：

```c++
int main(int argc, char** argv)
{      ...
        ProcessState::self()->startThreadPool();
        IPCThreadState::self()->joinThreadPool();
 }
```

可以看出Android APP上层应用的进程一般是开启一个Binder线程，而对于SystemServer或者media服务等使用频率高，服务复杂的进程，一般都是开启两个或者更多。来看第二个问题，**Binder线程的数目是固定的吗？答案是否定的，**驱动会根据目标进程中是否存在足够多的Binder线程来告诉进程是不是要新建Binder线程，详细逻辑，首先看一下新建Binder线程的入口：

```c++
status_t IPCThreadState::executeCommand(int32_t cmd)
{
    BBinder* obj;
    RefBase::weakref_type* refs;
    status_t result = NO_ERROR;
    switch (cmd) {
    ...
    // 可以根据内核返回数据创建新的binder线程
    case BR_SPAWN_LOOPER:
        mProcess->spawnPooledThread(false);
        break;
}
```

executeCommand一定是从Bindr驱动返回的BR命令，这里是BR_SPAWN_LOOPER，什么时候，Binder驱动会向进程发送BR_SPAWN_LOOPER呢？全局搜索之后，发现只有一个地方binder_thread_read，**如果直观的想一下，什么时候需要新建Binder线程呢？很简单，不够用的时候**，注意上面使用的是spawnPooledThread(false)，也就是说这里启动的都是普通Binder线程。为了了解启动时机，先看一些binder_proc内部判定参数的意义：

```c++
struct binder_proc {
	...
	int max_threads; 				// 进程所能启动的最大非主Binder线程数目
	int requested_threads;			// 请求启动的非主线程数
	int requested_threads_started;//已经启动的非主线程数
	int ready_threads;				// 当前可用的Binder线程数
	...
};
```

再来看binder_thread_read函数中是么时候会去请求新建Binder线程，以Android APP进程为例子，通过前面的分析知道APP进程天然支持Binder通信，因为它有一个Binder主线程，启动之后就会阻塞等待Client请求,这里会更新proc->ready_threads，第一次阻塞等待的时候proc->ready_threads=1，之后睡眠。

```c++
binder_thread_read（）{
  ...
 retry:
    //当前线程todo队列为空且transaction栈为空，则代表该线程是空闲的 ，看看是不是自己被复用了
    wait_for_proc_work = thread->transaction_stack == NULL &&
        list_empty(&thread->todo);
 ...//可用线程个数+1
    if (wait_for_proc_work)
        proc->ready_threads++; 
    binder_unlock(__func__);
    if (wait_for_proc_work) {
        ...
            //当进程todo队列没有数据,则进入休眠等待状态
            ret = wait_event_freezable_exclusive(proc->wait, binder_has_proc_work(proc, thread));
    } else {
        if (non_block) {
            ...
        } else
            //当线程todo队列没有数据，则进入休眠等待状态
            ret = wait_event_freezable(thread->wait, binder_has_thread_work(thread));
    }    
    binder_lock(__func__);
    //被唤醒可用线程个数-1
    if (wait_for_proc_work)
        proc->ready_threads--; 
    thread->looper &= ~BINDER_LOOPER_STATE_WAITING;
    ...
    while (1) {
        uint32_t cmd;
        struct binder_transaction_data tr;
        struct binder_work *w;
        struct binder_transaction *t = NULL;
        
        //先考虑从线程todo队列获取事务数据
        if (!list_empty(&thread->todo)) {
            w = list_first_entry(&thread->todo, struct binder_work, entry);
        //线程todo队列没有数据, 则从进程todo对获取事务数据
        } else if (!list_empty(&proc->todo) && wait_for_proc_work) {
            w = list_first_entry(&proc->todo, struct binder_work, entry);
        } else {
        }
		 ..
        if (t->buffer->target_node) {
            cmd = BR_TRANSACTION;  //设置命令为BR_TRANSACTION
        } else {
            cmd = BR_REPLY; //设置命令为BR_REPLY
        }
        .. 
done:
    *consumed = ptr - buffer;
    //创建线程的条件
    if (proc->requested_threads + proc->ready_threads == 0 &&
        proc->requested_threads_started < proc->max_threads &&
        (thread->looper & (BINDER_LOOPER_STATE_REGISTERED |
         BINDER_LOOPER_STATE_ENTERED))) {
         //需要新建的数目线程数+1
        proc->requested_threads++;
        // 生成BR_SPAWN_LOOPER命令，用于创建新的线程
        put_user(BR_SPAWN_LOOPER, (uint32_t __user *)buffer)；
    }
    return 0;
}
```

被Client唤醒后proc->ready_threads会-1，之后变成0，这样在执行到done的时候，就会发现proc->requested_threads + proc->ready_threads == 0，这是新建Binder线程的一个必须条件，再看下其他几个条件

```c++
if (proc->requested_threads + proc->ready_threads == 0 &&
	        proc->requested_threads_started < proc->max_threads &&
	        (thread->looper & (BINDER_LOOPER_STATE_REGISTERED |
	         BINDER_LOOPER_STATE_ENTERED)))  
```

- proc->requested_threads + proc->ready_threads == 0 ：**如果目前还没申请新建Binder线程，并且proc->ready_threads空闲Binder线程也是0，就需要新建一个Binder线程，其实就是为了保证有至少有一个空闲的线程**。
- proc->requested_threads_started < proc->max_threads：**目前启动的普通Binder线程数requested_threads_started还没达到上限（默认APP进程是15）**
- thread->looper & (BINDER_LOOPER_STATE_REGISTERED | BINDER_LOOPER_STATE_ENTERED) 当先线程是Binder线程，这个是一定满足的，不知道为什么列出来

proc->max_threads是多少呢？不同的进程其实设置的是不一样的，看普通的APP进程，在ProcessState::self()新建ProcessState单利对象的时候会调用ioctl(fd, BINDER_SET_MAX_THREADS, &maxThreads);设置上限,可以看到默认设置的上限是15。

```c++
static int open_driver()
{
    int fd = open("/dev/binder", O_RDWR);
    ...
    size_t maxThreads = 15;
        result = ioctl(fd, BINDER_SET_MAX_THREADS, &maxThreads);
    ...
}
```

如果满足新建的条件，就会将proc->requested_threads加1，并在驱动执行完毕后，利用put_user(BR_SPAWN_LOOPER, (uint32_t __user *)buffer)；通知服务端在用户空间发起新建Binder线程的操作，新建的是普通Binder线程，最终再进入binder_thread_write的BC_REGISTER_LOOPER:

```c++
int binder_thread_write(struct binder_proc *proc, struct binder_thread *thread,
			void __user *buffer, int size, signed long *consumed)
  {
		...
		case BC_REGISTER_LOOPER:
			  ...
				// requested_threads -- 
				proc->requested_threads--;
				proc->requested_threads_started++;
			}
}
```

这里会将proc->requested_threads复原，其实就是-1，并且启动的Binder线程数+1。

个人理解，之所以采用动态新建Binder线程的意义有两点，第一：如果没有Client请求服务，就保持线程数不变，减少资源浪费，需要的时候再分配新线程。第二：有请求的情况下，保证至少有一个空闲线程是给Client端，以提高Server端响应速度。

## Client端线程睡眠在哪个队列上，唤醒Server端哪个等待队列上的线程

先看第一部分：发送端线程睡眠在哪个队列上？

**发送端线程一定睡眠在自己binder_thread的等待队列上，并且，该队列上有且只有自己一个睡眠线程**

再看第二部分：在Binder驱动去唤醒线程的时候，唤醒的是哪个等待队列上的线程？

理解这个问题需要理解binder_thread中的 struct binder_transaction * transaction_stack栈，这个栈规定了transaction的执行顺序：**栈顶的一定先于栈内执行**。

如果本地操作是BC_REPLY,一定是唤醒之前发送等待的线程，这个是100%的，但是如果是BC_TRANSACTION，那就不一定了,尤其是当两端互为服务相互请求的时候，场景如下：

- 进程A的普通线程AT1请求B进程的B1服务，唤醒B进程的Binder线程，AT1睡眠等待服务结束
- B进程的B1服务在执行的的时候，需要请求进程A的A1服务，则B进程的Binder线程BT1睡眠，唤醒A进程的Binder线程，等待服务结束
- A进程的A1服务在执行的时候，又需要B进程的B2服务，则A进程的binder线程AT2睡眠，唤醒B进程的Binder线程，等待服务结束

这个时候就会遇到一个问题：唤醒哪个线程比较合适？是睡眠在进程队列上的线程，还是之前睡眠的线程BT1？答案是：之前睡眠的线程BT1，具体看下面的图解分析

首先第一步A普通线程去请求B进程的B1服务，这个时候在A进程的AT1线程的binder_ref中会将binder_transaction1入栈，而同样B的Binder线程在读取binder_work之后，也会将binder_transaction1加入自己的堆栈，如下图：

![binder_transaction堆栈及唤醒那个队列1](media/format,png-20201231114140658.jpeg)

而当B的Binder线程被唤醒后，执行Binder实体中的服务时，发现服务函数需要反过来去请求A端的A1服务，那就需要通过Binder向A进程发送请求，并新建binder_transaction2压入自己的binder_transaction堆栈，而A进程的Binder线程被唤醒后也会将binder_transaction2加入自己的堆栈，会后效果如下：

![binder_transaction堆栈及唤醒那个队列2.jpg](media/format,png-20201231114140080.jpeg)

这个时候，还是没有任何问题，但是恰好在执行A1服务的时候，又需要请求B2服务，这个时候，A1线程重复上述压栈过程，新建binder_transaction3压入自己的栈，不过在写入到目标端B的时候，会面临一个抉择，写入那个队列，是binder_proc上的队列，还是正在等候A返回的BT1线程的队列？

![binder_transaction堆栈及唤醒那个队列3.jpg](media/format,png-20201231114140139.jpeg)

结果已经说过，是BT1的队列，为什么呢？因为BT1队列上的之前的binder_transaction2在等待A进程执行完，但是A端的binder_transaction3同样要等待binder_transaction3在B进程中执行完毕，也就是说，binder_transaction3在B端一定是先于binder_transaction2执行的，因此唤醒BT1线程，并将binder_transaction3压入BT2的栈，等binder_transaction3执行完毕，出栈后，binder_transaction2才能执行，这样，既不妨碍binder_transaction2的执行，同样也能让睡眠的BT1进程提高利用率，因为最终的堆栈效果就是：

![binder_transaction堆栈及唤醒那个队列4.jpg](media/format,png-20201231114140210.jpeg)

而当binder_transaction3完成，出栈的过程其实就简单了，

- BT1 执行binder_transaction3，唤醒A端AT2 Binder线程，并且BT1继续睡眠（因为还有等待的transaction）
- AT2 执行binder_transaction2，唤醒BT1
- BT1 执行binder_transaction1，唤醒AT1
- 执行结束

从这里可以看出，其实设计的还是很巧妙的，让线程复用，提高了效率，还避免了新建不必要的Binder线程，在binder驱动中岛实现代码，其实就是根据binder_transaction中堆栈记录查询，


```c++
static void binder_transaction(struct binder_proc *proc,
struct binder_thread *thread,
struct binder_transaction_data *tr, int reply)
{	…
	while (tmp) {
		// 找到对方正在等待自己进程的线程，如果线程没有在等待自己进程的返回，就不要找了
		// 判断是不target_proc中，是不是有线程，等待当前线程
		// thread->transaction_stack，这个时候，
		// 是binder线程的，不是普通线程 B去请求A服务，
		// 在A服务的时候，又请求了B，这个时候，A的服务一定要等B处理完，才能再返回B，可以放心用B
		if (tmp->from && tmp->from->proc == target_proc)
				target_thread = tmp->from;
				tmp = tmp->from_parent;
		  ...			
    	}
	} 
}
```

## Binder协议中BC与BR的区别

BC与BR主要是标志数据及Transaction流向，其中BC是从用户空间流向内核，而BR是从内核流线用户空间，比如Client向Server发送请求的时候，用的是BC_TRANSACTION，当数据被写入到目标进程后，target_proc所在的进程被唤醒，在内核空间中，会将BC转换为BR，并将数据与操作传递该用户空间。

<img src="media/format,png-20201231114140246.jpeg" alt="BR与BC区别" style="zoom:50%;" />

## Binder在传输数据的时候是如何层层封装的–不同层次使用的数据结构（命令的封装）

内核中，与用户空间对应的结构体对象都需要新建，但传输数据的数据只拷贝一次，就是一次拷贝的时候。

从Client端请求开始分析，暂不考虑java层，只考虑Native，以ServiceManager的addService为例，具体看一下

```c++
MediaPlayerService::instantiate();
```

MediaPlayerService会新建Binder实体，并将其注册到ServiceManager中:

```c++
void MediaPlayerService::instantiate() {
    defaultServiceManager()->addService(
            String16("media.player"), new MediaPlayerService());
}	
```

这里defaultServiceManager其实就是获取ServiceManager的远程代理：

```c++
sp<IServiceManager> defaultServiceManager()
{
    if (gDefaultServiceManager != NULL) return gDefaultServiceManager;
    
    {
        AutoMutex _l(gDefaultServiceManagerLock);
        if (gDefaultServiceManager == NULL) {
            gDefaultServiceManager = interface_cast<IServiceManager>(
                ProcessState::self()->getContextObject(NULL));
        }
    }
    
    return gDefaultServiceManager;
}
```

如果将代码简化其实就是

```c++
return gDefaultServiceManager = BpServiceManager (new BpBinder(0));
```

addService就是调用BpServiceManager的addService，

```c++
virtual status_t addService(const String16& name, const sp<IBinder>& service,
        bool allowIsolated)
{
    Parcel data, reply;
    data.writeInterfaceToken(IServiceManager::getInterfaceDescriptor());
    data.writeString16(name);
    data.writeStrongBinder(service);
    data.writeInt32(allowIsolated ? 1 : 0);
    status_t err = remote()->transact(ADD_SERVICE_TRANSACTION, data, &reply);
    return err == NO_ERROR ? reply.readExceptionCode() : err;
}
```

这里会开始第一步的封装，数据封装，其实就是讲具体的传输数据写入到Parcel对象中，与Parcel对应是ADD_SERVICE_TRANSACTION等具体操作。比较需要注意的就是data.writeStrongBinder，这里其实就是把Binder实体压扁：

```c++
status_t Parcel::writeStrongBinder(const sp<IBinder>& val)
{
    return flatten_binder(ProcessState::self(), val, this);
}
```

具体做法就是转换成flat_binder_object，以传递Binder的类型、指针之类的信息：

```c++
status_t flatten_binder(const sp<ProcessState>& proc,
    const sp<IBinder>& binder, Parcel* out)
{
    flat_binder_object obj;
    
    obj.flags = 0x7f | FLAT_BINDER_FLAG_ACCEPTS_FDS;
    if (binder != NULL) {
        IBinder *local = binder->localBinder();
        if (!local) {
            BpBinder *proxy = binder->remoteBinder();
            if (proxy == NULL) {
                ALOGE("null proxy");
            }
            const int32_t handle = proxy ? proxy->handle() : 0;
            obj.type = BINDER_TYPE_HANDLE;
            obj.handle = handle;
            obj.cookie = NULL;
        } else {
            obj.type = BINDER_TYPE_BINDER;
            obj.binder = local->getWeakRefs();
            obj.cookie = local;
        }
    } else {
        obj.type = BINDER_TYPE_BINDER;
        obj.binder = NULL;
        obj.cookie = NULL;
    }
    
    return finish_flatten_binder(binder, obj, out);
}
```

接下来看 remote()->transact(ADD_SERVICE_TRANSACTION, data, &reply); 在上面的环境中，remote()函数返回的就是BpBinder(0)，

```c++
status_t BpBinder::transact(
    uint32_t code, const Parcel& data, Parcel* reply, uint32_t flags)
{
    // Once a binder has died, it will never come back to life.
    if (mAlive) {
        status_t status = IPCThreadState::self()->transact(
            mHandle, code, data, reply, flags);
        if (status == DEAD_OBJECT) mAlive = 0;
        return status;
    }

    return DEAD_OBJECT;
}
```

之后通过 IPCThreadState::self()->transact( mHandle, code, data, reply, flags)进行进一步封装:

```c++
status_t IPCThreadState::transact(int32_t handle,
				uint32_t code, const Parcel& data,
				Parcel* reply, uint32_t flags){
	if ((flags & TF_ONE_WAY) == 0) {
		if (err == NO_ERROR) {
			err = writeTransactionData(BC_TRANSACTION, flags, handle, code, data, NULL);
		}
		if (reply) {
			err = waitForResponse(reply);
		} 
		..
	return err;
	}
```

writeTransactionData(BC_TRANSACTION, flags, handle, code, data, NULL);是进一步封装的入口，在这个函数中Parcel& data、handle、code、被进一步封装成binder_transaction_data对象，并拷贝到mOut的data中去，同时也会将BC_TRANSACTION命令也写入mOut，这里与binder_transaction_data对应的CMD是BC_TRANSACTION，binder_transaction_data也存储了数据的指引新信息：

```c++
status_t IPCThreadState::writeTransactionData(int32_t cmd, uint32_t binderFlags,
    int32_t handle, uint32_t code, const Parcel& data, status_t* statusBuffer)
{
    binder_transaction_data tr;
    tr.target.handle = handle;
    tr.code = code;
    tr.flags = binderFlags;
    tr.cookie = 0;
    tr.sender_pid = 0;
    tr.sender_euid = 0;
    const status_t err = data.errorCheck();
    if (err == NO_ERROR) {
        tr.data_size = data.ipcDataSize();
        tr.data.ptr.buffer = data.ipcData();
        tr.offsets_size = data.ipcObjectsCount()*sizeof(size_t);
        tr.data.ptr.offsets = data.ipcObjects();
    } ..
    mOut.writeInt32(cmd);
    mOut.write(&tr, sizeof(tr));
    return NO_ERROR;
}
```

mOut封装结束后，会通过waitForResponse调用talkWithDriver继续封装：

```c++
status_t IPCThreadState::talkWithDriver(bool doReceive)
{
    binder_write_read bwr;
    // Is the read buffer empty? 这里会有同时返回两个命令的情况 BR_NOOP、BR_COMPLETE
    const bool needRead = mIn.dataPosition() >= mIn.dataSize();
    // We don't want to write anything if we are still reading
    // from data left in the input buffer and the caller
    // has requested to read the next data.
    const size_t outAvail = (!doReceive || needRead) ? mOut.dataSize() : 0;
    bwr.write_size = outAvail;
    bwr.write_buffer = (long unsigned int)mOut.data();	    // This is what we'll read.
    if (doReceive && needRead) {
        bwr.read_size = mIn.dataCapacity();
        bwr.read_buffer = (long unsigned int)mIn.data();
    } else {
        bwr.read_size = 0;
        bwr.read_buffer = 0;
    }
    // Return immediately if there is nothing to do.
    if ((bwr.write_size == 0) && (bwr.read_size == 0)) return NO_ERROR;
    bwr.write_consumed = 0;
    bwr.read_consumed = 0;
    status_t err;
    do {
        。。
        if (ioctl(mProcess->mDriverFD, BINDER_WRITE_READ, &bwr) >= 0)
            err = NO_ERROR；
        if (mProcess->mDriverFD <= 0) {
            err = -EBADF;
        }
    } while (err == -EINTR);

    if (err >= NO_ERROR) {
        if (bwr.write_consumed > 0) {
            if (bwr.write_consumed < (ssize_t)mOut.dataSize())
                mOut.remove(0, bwr.write_consumed);
            else
                mOut.setDataSize(0);
        }
        if (bwr.read_consumed > 0) {
            mIn.setDataSize(bwr.read_consumed);
            mIn.setDataPosition(0);
        }
        return NO_ERROR;
    }
    return err;
}
```

talkWithDriver会将mOut中的数据与命令继续封装成binder_write_read对象，其中bwr.write_buffer就是mOut中的data（binder_transaction_data+BC_TRRANSACTION），之后就会通过ioctl与binder驱动交互，进入内核，这里与binder_write_read对象对应的CMD是BINDER_WRITE_READ，进入驱动后，是先写后读的顺序，所以才叫BINDER_WRITE_READ命令，与BINDER_WRITE_READ层级对应的几个命令码一般都是跟线程、进程、数据整体传输相关的操作，不涉及具体的业务处理,比如BINDER_SET_CONTEXT_MGR是将线程编程ServiceManager线程，并创建0号Handle对应的binder_node、BINDER_SET_MAX_THREADS是设置最大的非主Binder线程数，而BINDER_WRITE_READ就是表示这是一次读写操作：

```c++
#define BINDER_CURRENT_PROTOCOL_VERSION 7
#define BINDER_WRITE_READ _IOWR('b', 1, struct binder_write_read)
#define BINDER_SET_IDLE_TIMEOUT _IOW('b', 3, int64_t)
#define BINDER_SET_MAX_THREADS _IOW('b', 5, size_t)
/* WARNING: DO NOT EDIT, AUTO-GENERATED CODE - SEE TOP FOR INSTRUCTIONS */
#define BINDER_SET_IDLE_PRIORITY _IOW('b', 6, int)
#define BINDER_SET_CONTEXT_MGR _IOW('b', 7, int)
#define BINDER_THREAD_EXIT _IOW('b', 8, int)
#define BINDER_VERSION _IOWR('b', 9, struct binder_version)
```

详细看一下binder_ioctl对于BINDER_WRITE_READ的处理，

```c++
static long binder_ioctl(struct file *filp, unsigned int cmd, unsigned long arg)
{
	switch (cmd) {
	case BINDER_WRITE_READ: {
		struct binder_write_read bwr;
		..
		<!--拷贝binder_write_read对象到内核空间-->
		if (copy_from_user(&bwr, ubuf, sizeof(bwr))) {
			ret = -EFAULT;
			goto err;
		}
		<!--根据是否需要写数据处理是不是要写到目标进程中去-->
		if (bwr.write_size > 0) {
			ret = binder_thread_write(proc, thread, (void __user *)bwr.write_buffer, bwr.write_size, &bwr.write_consumed);
		}
	  <!--根据是否需要写数据处理是不是要读，往自己进程里读数据-->
		if (bwr.read_size > 0) {
			ret = binder_thread_read(proc, thread, (void __user *)bwr.read_buffer, bwr.read_size, &bwr.read_consumed, filp->f_flags & O_NONBLOCK);
			<!--是不是要同时唤醒进程上的阻塞队列-->
			if (!list_empty(&proc->todo))
				wake_up_interruptible(&proc->wait);
		}
		break;
	}
	case BINDER_SET_MAX_THREADS:
		if (copy_from_user(&proc->max_threads, ubuf, sizeof(proc->max_threads))) {
		}
		break;
	case BINDER_SET_CONTEXT_MGR:
	   .. break;
	case BINDER_THREAD_EXIT:
		binder_free_thread(proc, thread);
		thread = NULL;
		break;
	case BINDER_VERSION:
	..
}
```

binder_thread_write(proc, thread, (void __user *)bwr.write_buffer, bwr.write_size, &bwr.write_consumed)这里其实就是把解析的binder_write_read对象再剥离，**bwr.write_buffer** 就是上面的（BC_TRANSACTION+ binder_transaction_data），

```c++
int binder_thread_write(struct binder_proc *proc, struct binder_thread *thread,
			void __user *buffer, int size, signed long *consumed)
{
	uint32_t cmd;
	void __user *ptr = buffer + *consumed;
	void __user *end = buffer + size;
	while (ptr < end && thread->return_error == BR_OK) {

		// binder_transaction_data  BC_XXX+binder_transaction_data
		if (get_user(cmd, (uint32_t __user *)ptr))  (BC_TRANSACTION)
			return -EFAULT;
		ptr += sizeof(uint32_t);
		switch (cmd) {
		..
		case BC_FREE_BUFFER: {
			...
		}
		case BC_TRANSACTION:
		case BC_REPLY: {
			struct binder_transaction_data tr;
			if (copy_from_user(&tr, ptr, sizeof(tr)))
				return -EFAULT;
			ptr += sizeof(tr);
			binder_transaction(proc, thread, &tr, cmd == BC_REPLY);
			break;
		}
		case BC_REGISTER_LOOPER:
			..
		case BC_ENTER_LOOPER:
			...
			thread->looper |= BINDER_LOOPER_STATE_ENTERED;
			break;
		case BC_EXIT_LOOPER:
		// 这里会修改读取的数据，
		*consumed = ptr - buffer;
	}
	return 0;
}
```

binder_thread_write会进一步根据CMD剥离出binder_transaction_data tr，交给binder_transaction处理，其实到binder_transaction数据几乎已经剥离极限，剩下的都是业务相关的，但是这里牵扯到一个Binder实体与Handle的转换过程，同城也牵扯两个进程在内核空间共享一些数据的问题，因此这里又进行了一次进一步的封装与拆封装，这里新封装了连个对象 binder_transaction与binder_work，有所区别的是binder_work可以看做是进程私有，但是binder_transaction是两个交互的进程共享的：binder_work是插入到线程或者进程的work todo队列上去的:

```c++
struct binder_thread {
	struct binder_proc *proc;
	struct rb_node rb_node;
	int pid;
	int looper;
	struct binder_transaction *transaction_stack;
	struct list_head todo;
	uint32_t return_error; /* Write failed, return error code in read buf */
	uint32_t return_error2; /* Write failed, return error code in read */
	wait_queue_head_t wait;
	struct binder_stats stats;
};
```

这里主要关心一下**binder_transaction**：binder_transaction主要记录了当前transaction的来源，去向，同时也为了返回做准备，buffer字段是一次拷贝后数据在Binder的内存地址。

```c++
struct binder_transaction {
	int debug_id;
	struct binder_work work;
	struct binder_thread *from; 
	struct binder_transaction *from_parent;
	struct binder_proc *to_proc;
	struct binder_thread *to_thread;
	struct binder_transaction *to_parent;
	unsigned need_reply:1;
	/* unsigned is_dead:1; */	/* not used at the moment */
	struct binder_buffer *buffer;
	unsigned int	code;
	unsigned int	flags;
	long	priority;
	long	saved_priority;
	uid_t	sender_euid;
};
```

binder_transaction函数主要负责的工作：

- 新建binder_transaction对象，并插入到自己的binder_transaction堆栈中

- 新建binder_work对象，插入到目标队列

- Binder与Handle的转换 (flat_binder_object)

   ```c++
     static void binder_transaction(struct binder_proc *proc,
     			       struct binder_thread *thread,
     			       struct binder_transaction_data *tr, int reply)
     {
     	struct binder_transaction *t;
     	struct binder_work *tcomplete;
     	size_t *offp, *off_end;
     	struct binder_proc *target_proc;
     	struct binder_thread *target_thread = NULL;
     	struct binder_node *target_node = NULL;
      **关键点1** 
     if (reply) {
     	in_reply_to = thread->transaction_stack;
     	thread->transaction_stack = in_reply_to->to_parent;
     	target_thread = in_reply_to->from;
     	target_proc = target_thread->proc;
     	}else {
     	if (tr->target.handle) {
     	    struct binder_ref * ref;
     			ref = binder_get_ref(proc, tr->target.handle);
     			target_node = ref->node;
     		} else {
     			target_node = binder_context_mgr_node;
     		}
     	  ..。
     **关键点2**
    	 t = kzalloc(sizeof( * t), GFP_KERNEL); 
    	 ...
    	 tcomplete = kzalloc(sizeof(*tcomplete), GFP_KERNEL);
    	 
    **关键点3 **
     off_end = (void *)offp + tr->offsets_size;
     
     for (; offp < off_end; offp++) {
     	struct flat_binder_object *fp;
     	fp = (struct flat_binder_object *)(t->buffer->data + *offp);
     	switch (fp->type) {
     	case BINDER_TYPE_BINDER:
     	case BINDER_TYPE_WEAK_BINDER: {
     		struct binder_ref *ref;
     		struct binder_node *node = binder_get_node(proc, fp->binder);
     		if (node == NULL) {
     			node = binder_new_node(proc, fp->binder, fp->cookie);
     		}..
     		ref = (target_proc, node);				   if (fp->type == BINDER_TYPE_BINDER)
     			fp->type = BINDER_TYPE_HANDLE;
     		else
     			fp->type = BINDER_TYPE_WEAK_HANDLE;
     		fp->handle = ref->desc;
     	} break;
     	case BINDER_TYPE_HANDLE:
     	case BINDER_TYPE_WEAK_HANDLE: {
     		struct binder_ref *ref = binder_get_ref(proc, fp->handle);
     		if (ref->node->proc == target_proc) {
     			if (fp->type == BINDER_TYPE_HANDLE)
     				fp->type = BINDER_TYPE_BINDER;
     			else
     				fp->type = BINDER_TYPE_WEAK_BINDER;
     			fp->binder = ref->node->ptr;
     			fp->cookie = ref->node->cookie;
     		} else {
     			struct binder_ref *new_ref;
     			new_ref = binder_get_ref_for_node(target_proc, ref->node);
     			fp->handle = new_ref->desc;
     		}
     	} break;
     	
    **关键点4** 将binder_work 插入到目标队列
    
   	t->work.type = BINDER_WORK_TRANSACTION;
     list_add_tail(&t->work.entry, target_list);
     tcomplete->type = BINDER_WORK_TRANSACTION_COMPLETE;
     list_add_tail(&tcomplete->entry, &thread->todo);
     if (target_wait)
     	wake_up_interruptible(target_wait);
     return;
   }
   ```

关键点1，找到目标进程，关键点2 创建binder_transaction与binder_work，关键点3 处理Binder实体与Handle转化，关键点4，将binder_work插入目标队列，并唤醒相应的等待队列，在处理Binder实体与Handle转化的时候，有下面几点注意的：

- 第一次注册Binder实体的时候，是向别的进程注册的，ServiceManager，或者SystemServer中的AMS服务
- Client请求服务的时候，一定是由Binder驱动为Client分配binder_ref，如果本进程的线程请求，fp->type = BINDER_TYPE_BINDER，否则就是fp->type = BINDER_TYPE_HANDLE。
- Android中的Parcel里面的对象一定是flat_binder_object

如此下来，写数据的流程所经历的数据结构就完了。再简单看一下被唤醒一方的读取流程，读取从阻塞在内核态的binder_thread_read开始，以传递而来的BC_TRANSACTION为例，binder_thread_read会根据一些场景添加BRXXX参数，标识驱动传给用户空间的数据流向：

```c++
enum BinderDriverReturnProtocol {

 BR_ERROR = _IOR_BAD('r', 0, int),
 BR_OK = _IO('r', 1),
 BR_TRANSACTION = _IOR_BAD('r', 2, struct binder_transaction_data),
 BR_REPLY = _IOR_BAD('r', 3, struct binder_transaction_data),

 BR_ACQUIRE_RESULT = _IOR_BAD('r', 4, int),
 BR_DEAD_REPLY = _IO('r', 5),
 BR_TRANSACTION_COMPLETE = _IO('r', 6),
 BR_INCREFS = _IOR_BAD('r', 7, struct binder_ptr_cookie),

 BR_ACQUIRE = _IOR_BAD('r', 8, struct binder_ptr_cookie),
 BR_RELEASE = _IOR_BAD('r', 9, struct binder_ptr_cookie),
 BR_DECREFS = _IOR_BAD('r', 10, struct binder_ptr_cookie),
 BR_ATTEMPT_ACQUIRE = _IOR_BAD('r', 11, struct binder_pri_ptr_cookie),

 BR_NOOP = _IO('r', 12),
 BR_SPAWN_LOOPER = _IO('r', 13),
 BR_FINISHED = _IO('r', 14),
 BR_DEAD_BINDER = _IOR_BAD('r', 15, void *),

 BR_CLEAR_DEATH_NOTIFICATION_DONE = _IOR_BAD('r', 16, void *),
 BR_FAILED_REPLY = _IO('r', 17),
};
```

之后，read线程根据binder_transaction新建binder_transaction_data对象，再通过copy_to_user，传递给用户空间，

```c++
static int
binder_thread_read(struct binder_proc *proc, struct binder_thread *thread,
	void  __user *buffer, int size, signed long *consumed, int non_block)
{
	while (1) {
			uint32_t cmd;
		 struct binder_transaction_data tr ;
			struct binder_work *w;
			struct binder_transaction *t = NULL;

		if (!list_empty(&thread->todo))
			w = list_first_entry(&thread->todo, struct binder_work, entry);
		else if (!list_empty(&proc->todo) && wait_for_proc_work)
			w = list_first_entry(&proc->todo, struct binder_work, entry);
		else {
			if (ptr - buffer == 4 && !(thread->looper & BINDER_LOOPER_STATE_NEED_RETURN)) /* no data added */
				goto retry;
			break;
		}
		
	// 数据大小
		tr.data_size = t->buffer->data_size;
		tr.offsets_size = t->buffer->offsets_size;
	// 偏移地址要加上
		tr.data.ptr.buffer = (void *)t->buffer->data + proc->user_buffer_offset;
		tr.data.ptr.offsets = tr.data.ptr.buffer + ALIGN(t->buffer->data_size, sizeof(void *));
	// 写命令
		if (put_user(cmd, (uint32_t __user *)ptr))
			return -EFAULT;
		// 写数据结构体到用户空间，
		ptr += sizeof(uint32_t);
		if (copy_to_user(ptr, &tr, sizeof(tr)))
			return -EFAULT;
		ptr += sizeof(tr);
}
```

上层通过ioctrl等待的函数被唤醒,假设现在被唤醒的是服务端，一般会执行请求,这里首先通过Parcel的ipcSetDataReference函数将数据将数据映射到Parcel对象中，之后再通过BBinder的transact函数处理具体需求；

```c++
status_t IPCThreadState::executeCommand(int32_t cmd)
{
	...
    // read到了数据请求，这里是需要处理的逻辑 ，处理完毕，
    case BR_TRANSACTION:
        {
            binder_transaction_data tr;
            Parcel buffer;
            buffer.ipcSetDataReference(
                reinterpret_cast<const uint8_t*>(tr.data.ptr.buffer),
                tr.data_size,
                reinterpret_cast<const size_t*>(tr.data.ptr.offsets),
                tr.offsets_size/sizeof(size_t), freeBuffer, this);
     ...
 // 这里是处理 如果非空，就是数据有效，
    if (tr.target.ptr) {
        // 这里什么是tr.cookie
        sp<BBinder> b((BBinder*)tr.cookie);
        const status_t error = b->transact(tr.code, buffer, &reply, tr.flags);
        if (error < NO_ERROR) reply.setError(error);

    }
```

这里的 b->transact(tr.code, buffer, &reply, tr.flags);就同一开始Client调用transact( mHandle, code, data, reply, flags)函数对应的处理类似，进入相对应的业务逻辑。

![Binder在传输数据的时候是如何层层封装的--不同层次使用的数据结构（命令的封装.jpg](media/format,png-20201231114140316.jpeg)

## Binder驱动传递数据的释放（释放时机）

在Binder通信的过程中，数据是从发起通信进程的用户空间直接写到目标进程内核空间，而这部分数据是直接映射到用户空间，必须等用户空间使用完数据才能释放，也就是说**Binder通信中内核数据的释放时机应该是用户空间控制的**，内种中释放内存空间的函数是binder_free_buf，其他的数据结构其实可以直接释放掉，执行这个函数的命令是BC_FREE_BUFFER。上层用户空间常用的入口是IPCThreadState::freeBuffer：

```c++
void IPCThreadState::freeBuffer(Parcel* parcel, const uint8_t* data, size_t dataSize,
                                const size_t* objects, size_t objectsSize,
                                void* cookie)
{
    if (parcel != NULL) parcel->closeFileDescriptors();
    IPCThreadState* state = self();
    state->mOut.writeInt32(BC_FREE_BUFFER);
    state->mOut.writeInt32((int32_t)data);
}
```

那什么时候会调用这个函数呢？在之前分析数据传递的时候，有一步是将binder_transaction_data中的数据映射到Parcel中去，其实这里是关键

```c++
status_t IPCThreadState::waitForResponse(Parcel *reply, status_t *acquireResult)
{
    int32_t cmd;
    int32_t err;

    while (1) {
    ...
        case BR_REPLY:
            {
            binder_transaction_data tr;
            // 注意这里是没有传输数据拷贝的，只有一个指针跟数据结构的拷贝，
            err = mIn.read(&tr, sizeof(tr));
            ALOG_ASSERT(err == NO_ERROR, "Not enough command data for brREPLY");
            if (err != NO_ERROR) goto finish;
            // free buffer，先设置数据，直接
            if (reply) {
                if ((tr.flags & TF_STATUS_CODE) == 0) {
                    // 牵扯到数据利用，与内存释放
                    reply->ipcSetDataReference(
                        reinterpret_cast<const uint8_t*>(tr.data.ptr.buffer),
                        tr.data_size,
                        reinterpret_cast<const size_t*>(tr.data.ptr.offsets),
                        tr.offsets_size/sizeof(size_t),
                        freeBuffer, this);
```

Parcel 的ipcSetDataReference函数不仅仅能讲数据映射到Parcel对象，同时还能将数据的清理函数映射进来

```c++
void Parcel::ipcSetDataReference(const uint8_t* data, size_t dataSize,
    const size_t* objects, size_t objectsCount, release_func relFunc, void* relCookie)
```

看函数定义中的release_func relFunc参数，这里就是指定内存释放函数，这里指定了IPCThreadState::freeBuffer函数，在Native层，Parcel在使用完，并走完自己的生命周期后，就会调用自己的析构函数，在其析构函数中调用了freeDataNoInit()，这个函数会间接调用上面设置的内存释放函数：

```c++
Parcel::~Parcel()
{
    freeDataNoInit();
}
```

这就是数据释放的入口，进入内核空间后，执行binder_free_buf，将这次分配的内存释放，同时更新binder_proc的binder_buffer表，重新标记那些内存块被使用了，哪些没被使用。

```c++
static void binder_free_buf(struct binder_proc *proc,
			    struct binder_buffer *buffer)
{
	size_t size, buffer_size;
	buffer_size = binder_buffer_size(proc, buffer);
	size = ALIGN(buffer->data_size, sizeof(void *)) +
		ALIGN(buffer->offsets_size, sizeof(void *));
	binder_debug(BINDER_DEBUG_BUFFER_ALLOC,
		     "binder: %d: binder_free_buf %p size %zd buffer"
		     "_size %zd\n", proc->pid, buffer, size, buffer_size);

	if (buffer->async_transaction) {
		proc->free_async_space += size + sizeof(struct binder_buffer);
		binder_debug(BINDER_DEBUG_BUFFER_ALLOC_ASYNC,
			     "binder: %d: binder_free_buf size %zd "
			     "async free %zd\n", proc->pid, size,
			     proc->free_async_space);
	}
	binder_update_page_range(proc, 0,
		(void *)PAGE_ALIGN((uintptr_t)buffer->data),
		(void *)(((uintptr_t)buffer->data + buffer_size) & PAGE_MASK),
		NULL);
	rb_erase(&buffer->rb_node, &proc->allocated_buffers);
	buffer->free = 1;
	if (!list_is_last(&buffer->entry, &proc->buffers)) {
		struct binder_buffer *next = list_entry(buffer->entry.next,
						struct binder_buffer, entry);
		if (next->free) {
			rb_erase(&next->rb_node, &proc->free_buffers);
			binder_delete_free_buffer(proc, next);
		}
	}
	if (proc->buffers.next != &buffer->entry) {
		struct binder_buffer *prev = list_entry(buffer->entry.prev,
						struct binder_buffer, entry);
		if (prev->free) {
			binder_delete_free_buffer(proc, buffer);
			rb_erase(&prev->rb_node, &proc->free_buffers);
			buffer = prev;
		}
	}
	binder_insert_free_buffer(proc, buffer);
}
```

Java层类似，通过JNI调用Parcel的freeData()函数释放内存，在用户空间，每次执行BR_TRANSACTION或者BR_REPLY，都会利用freeBuffer发送请求，去释放内核中的内存

## 简单的Binder通信C/S模型

<img src="media/format,png-20201231114140255.jpeg" alt="简单的Binder通信模型" style="zoom:67%;" />

很多文章将Binder框架定义了四个角色：Server，Client，ServiceManager、以及Binder驱动，但这容易将人引导到歧途：好像所有的Binder服务都需要去ServiceManager去注册才能使用，其实不是这样。例如，平时APP开发通过bindService启动的服务，以及有些自己定义的AIDL远程调用，都不一定都ServiceManager注册这条路，**个人理解：ServiceManager主要功能是：管理系统服务，比如AMS、WMS、PKMS服务等**，而APP通过的bindService启动的Binder服务其实是由SystemServer的ActivityManagerService负责管理。这篇主要关注Android APP Java层Binder通信一些奇葩点：

- ServiceManager addService的限制（并非服务都能使用ServiceManager的addService）
- bindService启动Service与Binder服务实体的流程
- Java层Binder实体与与BinderProxy是如何实例化及使用的，与Native层的关系是怎样的
- Parcel readStrongBinder与writeStrongBinder的原理（首先两端知晓）
- 数据传递参数AIDL单向与双向（inout）

## ServiceManager addService的限制–并非服务都能通过addService添加到ServiceManager

ServiceManager其实主要的面向对象是系统服务，大部分系统服务都是由SystemServer进程总添加到ServiceManager中去的，在通过ServiceManager添加服务的时候，是有些权限校验的，源码如下：

```c++
int svc_can_register(unsigned uid, uint16_t *name)
 {
    unsigned n;
    // 谁有权限add_service 0进程，或者 AID_SYSTEM进程
    if ((uid == 0) || (uid == AID_SYSTEM))
        return 1;
	 for (n = 0; n < sizeof(allowed) / sizeof(allowed[0]); n++)
        if ((uid == allowed[n].uid) && str16eq(name, allowed[n].name))
            return 1;
    return 0;
}
```

可以看到 (uid == 0) 或者 (uid == AID_SYSTEM)的进程都是可以添加服务的，uid=0，代表root用户，而uid=AID_SYSTEM，代表系统用户 。或者是一些特殊的配置进程。SystemServer进程在被Zygote创建的时候，就被分配了UID 是AID_SYSTEM（1000），

```c++
private static boolean startSystemServer()
        throws MethodAndArgsCaller, RuntimeException {
    /* Hardcoded command line to start the system server */
    String args[] = {
        "--setuid=1000",
        "--setgid=1000",
        "--setgroups=1001,1002,1003,1004,1005,1006,1007,1008,1009,1010,1018,3001,3002,3003,3006,3007",
        "--capabilities=130104352,130104352",
        "--runtime-init",
        "--nice-name=system_server",
        "com.android.server.SystemServer",
    };
```

Android每个APP的UID，都是不同的，用了Linux的UID那一套，但是没完全沿用，这里不探讨，**总之，普通的进程是没有权限注册到ServiceManager中的，那么APP平时通过bindService启动的服务怎么注册于查询的呢？接管这个任务的就是SystemServer的ActivityManagerService**。

## bindService启动Service与Binder服务实体的流程 （ActivityManagerService）

- bindService的框架
- binder服务实例化与转化
- 业务逻辑的唤醒
- 请求代理的转化与唤醒

bindService比startService多了一套Binder通信，其余的流程基本相同，而startService的流程，同startActivity差不多，四大组件的启动流程这里不做分析点，主要看bindService中C/S通信的建立流程，在这个流程里面，APP与服务端互为C/S的特性更明显，在APP开发的时候，binder服务是通过Service来启动的。Service的启动方式有两种startService，与bindService，这里只考虑后者，另外启动的binder服务也分为两种情况：第一种，client同server位于同一进程，可以看做内部服务，第二种，Client与Server跨进程，即使是位于同一个APP，第一桶可以不用AIDL来编写，但是第二种必须通过AIDL实现跨进程通信，看一个最简单的AIDL例子，首先在定义一个aidl接口：

> IMyAidlInterface.aidl

```c++
interface IMyAidlInterface {
	void communicate(int count);
}
```

IMyAidlInterface.aidl定义了通信的借口，通过build之后，构建工具会自动为IMyAidlInterface.aidl生成一些辅助类，这些辅助类主要作用是生成Binder通信协议框架，必须保证两方通信需要指令相同，才能解析通信内容。天王盖地虎，宝塔镇河妖。Java层Binder的对应关系Binder与BinderProxy从这里可以看出，binder采用了代理模式 stub与proxy对应，使用aidl实现的服务时候，Client如果想要获得Binder实体的代理可以通过asInterface来处理，比如如果在同一进程就是实体，不在就新建代理对象

```java
	public interface IMyAidlInterface extends android.os.IInterface {

    public static abstract class Stub extends android.os.Binder implements com.snail.labaffinity.IMyAidlInterface {
        private static final java.lang.String DESCRIPTOR = "com.snail.labaffinity.IMyAidlInterface";

        public Stub() {
            this.attachInterface(this, DESCRIPTOR);
        }

        public static com.snail.labaffinity.IMyAidlInterface asInterface(android.os.IBinder obj) {
            if ((obj == null)) {
                return null;
            }
            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if (((iin != null) && (iin instanceof com.snail.labaffinity.IMyAidlInterface))) {
                return ((com.snail.labaffinity.IMyAidlInterface) iin);
            }
            return new com.snail.labaffinity.IMyAidlInterface.Stub.Proxy(obj);
        }

        @Override
        public android.os.IBinder asBinder() {
            return this;
        }

        @Override
        public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException {
            switch (code) {
                case INTERFACE_TRANSACTION: {
                    reply.writeString(DESCRIPTOR);
                    return true;
                }
                case TRANSACTION_communicate: {
                    data.enforceInterface(DESCRIPTOR);
                    int _arg0;
                    _arg0 = data.readInt();
                    this.communicate(_arg0);
                    reply.writeNoException();
                    return true;
                }
            }
            return super.onTransact(code, data, reply, flags);
        }

        private static class Proxy implements com.snail.labaffinity.IMyAidlInterface {
            private android.os.IBinder mRemote;

            Proxy(android.os.IBinder remote) {
                mRemote = remote;
            }

            @Override
            public android.os.IBinder asBinder() {
                return mRemote;
            }

            public java.lang.String getInterfaceDescriptor() {
                return DESCRIPTOR;
            }

            @Override
            public void communicate(int count) throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    _data.writeInt(count);
                    mRemote.transact(Stub.TRANSACTION_communicate, _data, _reply, 0);
                    _reply.readException();
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
            }
        }

        static final int TRANSACTION_communicate = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
    }

    public void communicate(int count) throws android.os.RemoteException;
}
```

启动Binder服务端封装Service，之所以成为封装Service，是因为Service对于Binder实体的最大作用是个作为新建服务的入口：

```java
public class AidlService extends Service {
    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return new BBinderService();
    }

    public class BBinderService extends IMyAidlInterface.Stub {

        @Override
        public void communicate(int count) throws RemoteException {
        }
    }
}
```

而启动的入口：

```java
public class MainActivity extends AppCompatActivity {
    ...
   void bind(){
    Intent intent = createExplicitFromImplicitIntent(MainActivity.this, new Intent("com.snail.labaffinity.service.AidlService"));
    bindService(intent, new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName componentName, IBinder iBinder) {
            IMyAidlInterface  iMyAidlInterface = IMyAidlInterface.Stub.asInterface(iBinder);
        }

        @Override
        public void onServiceDisconnected(ComponentName componentName) {
        }
    }, Context.BIND_AUTO_CREATE);
    }
 }
```

以上四个部分就组成了AIDL跨进程服务的基本组件，现在从ActivitybindService入口开始分析：bindService大部分的流程与startActivity类似，其实都是通过AMS启动组件，这里只将一些不同的地方，Activity启动只需要Intent就可以了，而Service的bind需要一个ServiceConnection对象，这个对象其实是为了AMS端在启动Service后回调用的，ServiceConnection是个接口，其实例在ContextImpl的：

```java
private boolean bindServiceCommon(Intent service, ServiceConnection conn, int flags,
                                  UserHandle user) {
    IServiceConnection sd;
    if (conn == null) {
        throw new IllegalArgumentException("connection is null");
    }
    if (mPackageInfo != null) {
        sd = mPackageInfo.getServiceDispatcher(conn, getOuterContext(),
                mMainThread.getHandler(), flags);
    } else {
        throw new RuntimeException("Not supported in system context");
    }
    validateServiceIntent(service);
    try {
        IBinder token = getActivityToken();
        if (token == null && (flags & BIND_AUTO_CREATE) == 0 && mPackageInfo != null
                && mPackageInfo.getApplicationInfo().targetSdkVersion
                < android.os.Build.VERSION_CODES.ICE_CREAM_SANDWICH) {
            flags |= BIND_WAIVE_PRIORITY;
        }
        service.prepareToLeaveProcess();
        int res = ActivityManagerNative.getDefault().bindService(
                mMainThread.getApplicationThread(), getActivityToken(), service,
                service.resolveTypeIfNeeded(getContentResolver()),
                sd, flags, getOpPackageName(), user.getIdentifier());
        if (res < 0) {
            throw new SecurityException(
                    "Not allowed to bind to service " + service);
        }
        return res != 0;
    } catch (RemoteException e) {
        throw new RuntimeException("Failure from system", e);
    }
}
```

mPackageInfo是一个LoadApk类，通过它的getServiceDispatcher获得一个IServiceConnection对象，这个对象一个Binder实体，看一下具体原理

```java
public final IServiceConnection getServiceDispatcher(ServiceConnection c,
        Context context, Handler handler, int flags) {
    synchronized (mServices) {
        LoadedApk.ServiceDispatcher sd = null;
        ArrayMap<ServiceConnection, LoadedApk.ServiceDispatcher> map = mServices.get(context);
        if (map != null) {
            sd = map.get(c);
        }
        if (sd == null) {
            sd = new ServiceDispatcher(c, context, handler, flags);
            if (map == null) {
                map = new ArrayMap<ServiceConnection, LoadedApk.ServiceDispatcher>();
                mServices.put(context, map);
            }
            map.put(c, sd);
        } else {
            sd.validate(context, handler);
        }
        return sd.getIServiceConnection();
    }
}
```

在LoadApk中IServiceConnection对象是通过context键值来存储ServiceDispatcher对象，而ServiceDispatcher对象内存会有个InnerConnection对象，该对象就是getServiceDispatcher的返回对象。因此bindServiceCommon最终调用
ActivityManagerNative.getDefault().bindService(x,x,x,x,x sd, x, x, x) 的时候，传递的参数sd其实就是一个InnerConnection对象，这是个Binder实体。但是，Binder.java中的Binder只是对native层BBinder的一个简单封装,真正的实例化还是通过JNI到native层去创建一个JavaBBinderHolder对象，并初始化gBinderOffsets，让其能映射Java层Binder对象，而JavaBBinderHolder中又可以实例化BBinder的实例JavaBBinder，不过BBinder的实例化时机并不在这里，而是在Parcel对象writeStrongBinder的时候，

```c++
static struct bindernative_offsets_t
{
    // Class state.
    jclass mClass;
    jmethodID mExecTransact;

    // Object state.
    jfieldID mObject;

} gBinderOffsets;

static void android_os_Binder_init(JNIEnv* env, jobject obj)
{
    JavaBBinderHolder* jbh = new JavaBBinderHolder();
    jbh->incStrong((void*)android_os_Binder_init);
    env->SetIntField(obj, gBinderOffsets.mObject, (int)jbh);
}
```

继续往下看bindService，会调用到ActivityManagerProxy的bindService

```java
public int bindService(IApplicationThread caller, IBinder token,
        Intent service, String resolvedType, IServiceConnection connection,
        int flags, int userId) throws RemoteException {
    Parcel data = Parcel.obtain();
    Parcel reply = Parcel.obtain();
    data.writeInterfaceToken(IActivityManager.descriptor);
    data.writeStrongBinder(caller != null ? caller.asBinder() : null);
    data.writeStrongBinder(token);
    service.writeToParcel(data, 0);
    data.writeString(resolvedType);
    data.writeStrongBinder(connection.asBinder());
    data.writeInt(flags);
    data.writeInt(userId);
    mRemote.transact(BIND_SERVICE_TRANSACTION, data, reply, 0);
    reply.readException();
    int res = reply.readInt();
    data.recycle();
    reply.recycle();
    return res;
}
```

利用Parcel的writeStrongBinder会将Binder实体写入到Parcel中去，这里首先看一下 Parcel data = Parcel.obtain();在java层Parcel只是一个容器，具体Parcel相关的操作都在Native层

```c++
static jint android_os_Parcel_create(JNIEnv* env, jclass clazz)
{
    Parcel* parcel = new Parcel();
    return reinterpret_cast<jint>(parcel);
}
```

这里的返回值，其实就是Parcel对象的地址，被赋值给了Parcel.java的mNativePtr成员变量，方便Native调用，接着看writeStrongBinder的实现，其实就是调用Parcel.cpp中的对应方法，通过flatten_binder将Binder实体对象打扁，创建flat_binder_object写入Parcel中，

```c++
static void android_os_Parcel_writeStrongBinder(JNIEnv* env, jclass clazz, jint nativePtr, jobject object)
{
    Parcel* parcel = reinterpret_cast<Parcel*>(nativePtr);
    if (parcel != NULL) {
        const status_t err = parcel->writeStrongBinder(ibinderForJavaObject(env, object));
        if (err != NO_ERROR) {
            signalExceptionForError(env, clazz, err);
        }
    }
}
```

ibinderForJavaObject主要为Java层Binder实例化native binder对象：

```c++
sp<IBinder> ibinderForJavaObject(JNIEnv* env, jobject obj)
{
    if (obj == NULL) return NULL;

    if (env->IsInstanceOf(obj, gBinderOffsets.mClass)) {
        JavaBBinderHolder* jbh = (JavaBBinderHolder*)
            env->GetIntField(obj, gBinderOffsets.mObject);
        return jbh != NULL ? jbh->get(env, obj) : NULL;
    }

    if (env->IsInstanceOf(obj, gBinderProxyOffsets.mClass)) {
        return (IBinder*)
            env->GetIntField(obj, gBinderProxyOffsets.mObject);
    }
    return NULL;
}
```

如果BBinder还没实例化，要通过JavaBBinderHolder的get函数实例化一个BBinder对象，这里就是**JavaBBinder**对象，综上分析Java层与Native的Binder其对应关系如下：

<img src="media/format,png-20201231114140361.png" alt="Java层Binder与native 层BBiner.png" style="zoom:50%;" />

BBinder对象被Parcel转换成flat_binder_object，经过一次拷贝写入目标进程，并执行BINDER_TYPE_BINDER与BINDER_TYPE_HANDLE的转换，如下：

```c++
static void
binder_transaction(struct binder_proc *proc, struct binder_thread *thread,
	struct binder_transaction_data *tr, int reply)
	...
 fp = (struct flat_binder_object *)(t->buffer->data + *offp);

	switch (fp->type) {
		case BINDER_TYPE_BINDER:
		case BINDER_TYPE_WEAK_BINDER: {..
			if (fp->type == BINDER_TYPE_BINDER)
				fp->type = BINDER_TYPE_HANDLE;
			else
				fp->type = BINDER_TYPE_WEAK_HANDLE;
			fp->handle = ref->desc;
		} break;
		case BINDER_TYPE_HANDLE:
		case BINDER_TYPE_WEAK_HANDLE: {..
			struct binder_ref *ref = binder_get_ref(proc, fp->handle);
			if (ref->node->proc == target_proc) {
				if (fp->type == BINDER_TYPE_HANDLE)
					fp->type = BINDER_TYPE_BINDER;
				else
					fp->type = BINDER_TYPE_WEAK_BINDER;
				fp->binder = ref->node->ptr;
				fp->cookie = ref->node->cookie;
			} else {
				struct binder_ref *new_ref;
				new_ref = binder_get_ref_for_node(target_proc, ref->node);
				fp->handle = new_ref->desc;
			}
		} break;
}
```

在内核中，bindService中的InnerConnection会由BINDER_TYPE_BINDER转换成BINDER_TYPE_HANDLE，之后，AMS线程被唤醒后，执行后面的流程，在前文分析Parcel数据转换的时候，在Binder线程被唤醒继续执行的时候，会将数据映射到一个natvie Parcel对象中

```c++
status_t IPCThreadState::executeCommand(int32_t cmd)
 {
    BBinder* obj;
    switch (cmd) {
     ..
    // read到了数据请求，这里是需要处理的逻辑 ，处理完毕，
    case BR_TRANSACTION:
        {
            binder_transaction_data tr;
            result = mIn.read(&tr, sizeof(tr));
            Parcel buffer;
            <!--关键点1 -->
            buffer.ipcSetDataReference(
                reinterpret_cast<const uint8_t*>(tr.data.ptr.buffer),
                tr.data_size,
                reinterpret_cast<const size_t*>(tr.data.ptr.offsets),
                tr.offsets_size/sizeof(size_t), freeBuffer, this);
            ...
          <!--关键点2 -->
        if (tr.target.ptr) {
            sp<BBinder> b((BBinder*)tr.cookie);
            const status_t error = b->transact(tr.code, buffer, &reply, tr.flags);
            if (error < NO_ERROR) reply.setError(error);
        }
        ..
      }
   }
```

首先看一下关键点1 ，这里将内核数据映射到一个用户空间的Parcel对象中去，之后在调用目标Service的transact函数，进而调用他的onTrasanct函数 , 通过前面的分析知道，Java层Binder在注册时候，最终注册的是JavaBBinder对象，看一下它的onTrasanct函数：

```c++
virtual status_t onTransact(
        uint32_t code, const Parcel& data, Parcel* reply, uint32_t flags = 0)
    {
        JNIEnv* env = javavm_to_jnienv(mVM);
        IPCThreadState* thread_state = IPCThreadState::self();
        const int strict_policy_before = thread_state->getStrictModePolicy();
        thread_state->setLastTransactionBinderFlags(flags);
        ..
        jboolean res = env->CallBooleanMethod(mObject, gBinderOffsets.mExecTransact,
            code, (int32_t)&data, (int32_t)reply, flags);
     	..
        return res != JNI_FALSE ? NO_ERROR : UNKNOWN_TRANSACTION;
    }
```

关键代码只有一句：env->CallBooleanMethod(mObject, gBinderOffsets.mExecTransact, code, (int32_t)&data, (int32_t)reply, flags)，其实就是调用Binder.java的execTransact函数，最终调用BBinder实例化类的onTransact

```c++
private boolean execTransact(int code, int dataObj, int replyObj,
            int flags) {
        Parcel data = Parcel.obtain(dataObj);
        Parcel reply = Parcel.obtain(replyObj);
        boolean res;
        try {
            res = onTransact(code, data, reply, flags);
        } ...
        reply.recycle();
        data.recycle();
        return res;
    }}
```

对于AMS而bindService对应的操作如下

```java
public boolean onTransact(int code, Parcel data, Parcel reply, int flags)
    throws RemoteException {
    。。
    case BIND_SERVICE_TRANSACTION: {
    data.enforceInterface(IActivityManager.descriptor);
    IBinder b = data.readStrongBinder();
    IApplicationThread app = ApplicationThreadNative.asInterface(b);
    IBinder token = data.readStrongBinder();
    Intent service = Intent.CREATOR.createFromParcel(data);
    String resolvedType = data.readString();
    b = data.readStrongBinder();
    int fl = data.readInt();
    int userId = data.readInt();
    IServiceConnection conn = IServiceConnection.Stub.asInterface(b);
    int res = bindService(app, token, service, resolvedType, conn, fl, userId);
    reply.writeNoException();
    reply.writeInt(res);
    return true;
}
```

b = data.readStrongBinder()会先读取Binder对象，这里会调用本地函数nativeReadStrongBinder(mNativePtr)，mNativePtr就是Native层Parcel的首地址：

```java
public final IBinder readStrongBinder() {
    return nativeReadStrongBinder(mNativePtr);
}
```

nativeReadStrongBinder(mNativePtr)会将本地Binder对象转化成Java层对象，其实就是将传输的InnerConnection读取出来，不过由于Binder驱动将BINDER_TYPE_BINDER转换成了BINDER_TYPE_HANDLE，对于AMS其实是实例化BinderProxy

```c++
static jobject android_os_Parcel_readStrongBinder(JNIEnv* env, jclass clazz, jint nativePtr)
{
    Parcel* parcel = reinterpret_cast<Parcel*>(nativePtr);
    if (parcel != NULL) {

        // /parcel->readStrongBinder() 其实就会创建BpBInder、
        return javaObjectForIBinder(env, parcel->readStrongBinder());
    }
    return NULL;
}
```

首先会利用Parcel.cpp的parcel->readStrongBinder()，读取binder对象，这里会根据flat_binder_object的类型，分别进行BBinder与BpBinder映射，如果是Binder实体直接将指针赋值out，如果不是，则根据handle获取或者新建BpBinder返回给out。

```c++
status_t unflatten_binder(const sp<ProcessState>& proc,
    const Parcel& in, sp<IBinder>* out)
{
    const flat_binder_object* flat = in.readObject(false);
    
    if (flat) {
        switch (flat->type) {
            case BINDER_TYPE_BINDER:
                *out = static_cast<IBinder*>(flat->cookie);
                return finish_unflatten_binder(NULL, *flat, in);
            case BINDER_TYPE_HANDLE:
                *out = proc->getStrongProxyForHandle(flat->handle);
                return finish_unflatten_binder(
                    static_cast<BpBinder*>(out->get()), *flat, in);
        }        
    }
    return BAD_TYPE;
}
```

之后会牵扯一个将native binder转换成java层Binder的操作，javaObjectForIBinder，这个函数很关键，是理解Java层BinderProxy或者BBinder实体的关键：

```c++
jobject javaObjectForIBinder(JNIEnv* env, const sp<IBinder>& val)
{
    if (val == NULL) return NULL;
    <!--关键点1-->
    if (val->checkSubclass(&gBinderOffsets)) {
        jobject object = static_cast<JavaBBinder*>(val.get())->object();
        return object;
    }
    AutoMutex _l(mProxyLock);
    <!--关键点2-->
    jobject object = (jobject)val->findObject(&gBinderProxyOffsets);
    if (object != NULL) {
        android_atomic_dec(&gNumProxyRefs);
        val->detachObject(&gBinderProxyOffsets);
        env->DeleteGlobalRef(object);
    }
    <!--关键点3-->
    object = env->NewObject(gBinderProxyOffsets.mClass, gBinderProxyOffsets.mConstructor);
    if (object != NULL) {
        env->SetIntField(object, gBinderProxyOffsets.mObject, (int)val.get());
        val->incStrong((void*)javaObjectForIBinder);
        jobject refObject = env->NewGlobalRef(
                env->GetObjectField(object, gBinderProxyOffsets.mSelf));
        val->attachObject(&gBinderProxyOffsets, refObject,
                jnienv_to_javavm(env), proxy_cleanup);
        sp<DeathRecipientList> drl = new DeathRecipientList;
        drl->incStrong((void*)javaObjectForIBinder);
        env->SetIntField(object, gBinderProxyOffsets.mOrgue, reinterpret_cast<jint>(drl.get()));
        android_atomic_inc(&gNumProxyRefs);
        incRefsCreated(env);
    }
    return object;
}
```

先看关键点1， checkSubclass默认返回false，但是JavaBBinder，该类对此函数进行了覆盖，如果是JavaBBinder，就会返回true，但入股是BpBinder，则会返回false，

```c++
bool checkSubclass(const void* subclassID) const
{
    return subclassID == &gBinderOffsets;
}
```

再看关键点2，如果是BpBinder，则需要首先在gBinderProxyOffsets中查找，是不是已经新建了Java层代理BinderProxy对象，如果没有，则新建即可，如果新建就看是否还存在缓存有效的BinderProxy。最后看关键点3 ：

```c++
env->NewObject(gBinderProxyOffsets.mClass, gBinderProxyOffsets.mConstructor)
```

其实就是新建BinderProxy对象，Java层的BinderProxy都是Native新建的，Java层并没有BinderProxy的新建入口，之后，再通过IServiceConnection.Stub.asInterface(b)进行转换，实例化一个IServiceConnection.Proxy代理对，该对象在Binder通信的基础上封装了业务逻辑，其实就是一些具体的操作。

```c++
 public static XXXAidlInterface asInterface(android.os.IBinder obj) {
            if ((obj == null)) {
                return null;
            }
            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if (((iin != null) && (iin instanceof XXXAidlInterface))) {
                return ((XXXAidlInterface) iin);
            }
            return new XXXAidlInterface.Stub.Proxy(obj);
        }
```

这里注意一点杜宇BinderProxy，obj.queryLocalInterface(DESCRIPTOR)返回为null，对于Binder实体，返回的是Binder自身，这样就能为上层区分出是生成代理还是存根自身，整体对象转换流程如下：

![ServiceConnection的回调作用](media/format,png-20201231114140435.png)

到这里分析了一半，Java层命令及回调Binder入口已经被传递给AMS，AMS之后需要负责启动Service，并通过回调入口为Client绑定服务，跟踪到AMS源码

```java
public int bindService(IApplicationThread caller, IBinder token,
        Intent service, String resolvedType,
        IServiceConnection connection, int flags, int userId) {
    ...
    synchronized(this) {
        return mServices.bindServiceLocked(caller, token, service, resolvedType,
                connection, flags, userId);
    }
}
```

最后调用ActiveService的bindServiceLocked，这里会分三中情况，

- Service已经经启动
- Service未启动，但是进程已经启动
- Service与进程君未启动

不过这里只讨论比较经典的Service未启动， Service未启动，但是进程已经启动的情况，关键代码如下

```java
 int bindServiceLocked(IApplicationThread caller, IBinder token,
            Intent service, String resolvedType,
            IServiceConnection connection, int flags, int userId) {

        try {
            .。。

            if ((flags&Context.BIND_AUTO_CREATE) != 0) {
                s.lastActivity = SystemClock.uptimeMillis();
          <!--关键点1-->
                if (bringUpServiceLocked(s, service.getFlags(), false) != null) {
                    return 0;
                }
            }
          <!--关键点2-->
           ..
           requestServiceBindingLocked(s, b.intent, false);
           ..
        }
}
```

ja关键点1其实就是启动Service，主要是通过ApplicationThread的binder通信通知App端启动Service，这个流程同Activity启动一样。关键点2是Service特有的：requestServiceBindingLocked，这个命令是告诉APP端：“在Service启动后需要向AMS发消息，之后AMS才能向其他需要绑定该Service的Client发送反馈”。

```java
AMS端
private final boolean requestServiceBindingLocked(ServiceRecord r,
        IntentBindRecord i, boolean rebind) {
    if ((!i.requested || rebind) && i.apps.size() > 0) {
       ..
          r.app.thread.scheduleBindService(r, i.intent.getIntent(), rebind);
       ..
      }        }
    return true;
}

 APP端
 private void handleBindService(BindServiceData data) {
    Service s = mServices.get(data.token);
    ...
    if (!data.rebind) {
        IBinder binder = s.onBind(data.intent);
        ActivityManagerNative.getDefault().publishService(
                data.token, data.intent, binder);
    }
}
```

ActivityManagerNative.getDefault().publishService会将启动的Binder服务实体传递给AMS，上面分析过Binder实体传输，这里的原理是一样的，AMS端在传输结束后，会获得Service端服务实体的引用，这个时候，就能通过最初的InnerConnection的回调将这个服务传递给Client端。Binder实体与引用的整体流程图如下：

![bindSerivce整体流程图](media/format,png-20201231114140501.png)

如果要深究Activity的bindService流程，可以按以下几步来分析

- 1、Activity调用bindService：通过Binder通知ActivityManagerService，要启动哪个Service
- 2、ActivityManagerService创建ServiceRecord，并利用ApplicationThreadProxy回调，通知APP新建并启动Service启动起来
- 3、ActivityManagerService把Service启动起来后，继续通过ApplicationThreadProxy，通知APP，bindService，其实就是让Service返回一个Binder对象给ActivityManagerService，以便AMS传递给Client
- 4、ActivityManagerService把从Service处得到这个Binder对象传给Activity，这里是通过IServiceConnection binder实现。
- 5、Activity被唤醒后通过Binder Stub的asInterface函数将Binder转换为代理Proxy，完成业务代理的转换，之后就能利用Proxy进行通信了。

![bindService流程](media/format,png-20201231114140471.png)

## 参考文档

* [Android Binder 分析——通信模型](http://light3moon.com/2015/01/28/Android Binder 分析——通信模型/)
* [理解 Binder 线程池的管理](https://gold.xitu.io/entry/58197bd9128fe10055a4a51e)
* [Android Binder 分析——多线程支持](http://light3moon.com/2015/01/28/Android Binder 分析——多线程支持/)
* [彻底理解Android Binder通信架构](http://gityuan.com/2016/09/04/binder-start-service/)
* [Android Binder机制の设计与实现4（Binder 协议）](http://blog.csdn.net/xujianqun/article/details/6677862)
* [Android四大组件与进程启动的关系](http://gityuan.com/2016/10/09/app-process-create-2/)
* [binder驱动-------之内存映射篇](http://blog.csdn.net/xiaojsj111/article/details/31422175)

## 来源

https://blog.csdn.net/happylishang/article/details/62234127