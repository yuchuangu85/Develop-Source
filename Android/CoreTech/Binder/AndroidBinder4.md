<h1 align="center">Android Binder原理（四）ServiceManager的启动过程</h1>

### **前言**

在上一篇文章中，我们以MediaPlayerService为例，讲解了系统服务是如何注册的（addService），既然有注册就势必要有获取，但是在了解获取服务前，我们最好先了解ServiceManager的启动过程，这样更有助于理解系统服务的注册和获取的过程。

另外还有一点需要说明的是，要想了解ServiceManager的启动过程，需要查看Kernel Binder部分的源码，这部分代码在内核源码中，AOSP源码是不包括内核源码的，因此需要单独下载，见[Android AOSP基础（二）AOSP源码和内核源码下载](http://liuwangshu.cn/framework/aosp/2-download-aosp.html)这篇文章。

### **1.ServiceManager的入口函数**

ServiceManager是init进程负责启动的，具体是在解析init.rc配置文件时启动的，init进程是在系统启动时启动的，因此ServiceManager亦是如此，不理解init进程和init.rc的可以看[Android系统启动流程（一）解析init进程启动过程](http://liuwangshu.cn/framework/booting/1-init.html)这篇文章。 rc文件内部由Android初始化语言编写（Android Init Language）编写的脚本，它主要包含五种类型语句：Action、Commands、Services、Options和Import。 在Android 7.0中对init.rc文件进行了拆分，每个服务一个rc文件。ServiceManager的启动脚本在servicemanager.rc中： frameworks/native/cmds/servicemanager/servicemanager.rc

```
service servicemanager /system/bin/servicemanager
    class core animation
    user system  //1
    group system readproc
    critical //2
    onrestart restart healthd  
    onrestart restart zygote
    onrestart restart audioserver
    onrestart restart media
    onrestart restart surfaceflinger
    onrestart restart inputflinger
    onrestart restart drm
    onrestart restart cameraserver
    onrestart restart keystore
    onrestart restart gatekeeperd
    writepid /dev/cpuset/system-background/tasks
    shutdown critical
```

service用于通知init进程创建名为servicemanager的进程，这个servicemanager进程执行程序的路径为/system/bin/servicemanager。 注释1的关键字user说明servicemanager是以用户system的身份运行的，注释2处的critical说明servicemanager是系统中的关键服务，关键服务是不会退出的，如果退出了，系统就会重启，当系统重启时就会启动用onrestart关键字修饰的进程，比如zygote、media、surfaceflinger等等。

servicemanager的入口函数在service_manager.c中: **frameworks/native/cmds/servicemanager/service_manager.c**

```
int main(int argc, char** argv)
{
    struct binder_state *bs;//1
    union selinux_callback cb;
    char *driver;

    if (argc > 1) {
        driver = argv[1];
    } else {
        driver = "/dev/binder";
    }

    bs = binder_open(driver, 128*1024);//2
    ...
    if (binder_become_context_manager(bs)) {//3
        ALOGE("cannot become context manager (%s)\n", strerror(errno));
        return -1;
    }
    ...
    if (getcon(&service_manager_context) != 0) {
        ALOGE("SELinux: Failed to acquire service_manager context. Aborting.\n");
        abort();
    }
    binder_loop(bs, svcmgr_handler);//4

    return 0;
}
```

注释1处的binder_state结构体用来存储binder的三个信息：

```
struct binder_state
{
    int fd; //binder设备的文件描述符
    void *mapped; //binder设备文件映射到进程的地址空间
    size_t mapsize; //内存映射后，系统分配的地址空间的大小，默认为128KB
};
```

main函数主要做了三件事： 1.注释2处调用binder_open函数用于打开binder设备文件，并申请128k字节大小的内存空间。 2.注释3处调用binder_become_context_manager函数，将servicemanager注册成为Binder机制的上下文管理者。 3.注释4处调用binder_loop函数，循环等待和处理client端发来的请求。

现在对这三件事分别进行讲解。

#### **1.1 打开binder设备**

binder_open函数用于打开binder设备文件，并且将它映射到进程的地址空间，如下所示。

**frameworks/native/cmds/servicemanager/binder.c**

```
struct binder_state *binder_open(const char* driver, size_t mapsize)
{
    struct binder_state *bs;
    struct binder_version vers;

    bs = malloc(sizeof(*bs));
    if (!bs) {
        errno = ENOMEM;
        return NULL;
    }

    bs->fd = open(driver, O_RDWR | O_CLOEXEC);//1
    if (bs->fd < 0) {
        fprintf(stderr,"binder: cannot open %s (%s)\n",
                driver, strerror(errno));
        goto fail_open;
    }
    //获取Binder的version
    if ((ioctl(bs->fd, BINDER_VERSION, &vers) == -1) ||
        (vers.protocol_version != BINDER_CURRENT_PROTOCOL_VERSION)) {//2
        fprintf(stderr,
                "binder: kernel driver version (%d) differs from user space version (%d)\n",
                vers.protocol_version, BINDER_CURRENT_PROTOCOL_VERSION);
        goto fail_open;
    }

    bs->mapsize = mapsize;
    bs->mapped = mmap(NULL, mapsize, PROT_READ, MAP_PRIVATE, bs->fd, 0);//3
    if (bs->mapped == MAP_FAILED) {
        fprintf(stderr,"binder: cannot map device (%s)\n",
                strerror(errno));
        goto fail_map;
    }
    return bs;

fail_map:
    close(bs->fd);
fail_open:
    free(bs);
    return NULL;
}
```

注释1处用于打开binder设备文件，后面会进行分析。 注释2处的ioctl函数用于获取Binder的版本，如果获取不到或者内核空间和用户空间的binder不是同一个版本就会直接goto到fail_open标签，释放binder的内存空间。 注释3处调用mmap函数进行内存映射，通俗来讲就是将binder设备文件映射到进程的地址空间，地址空间的大小为mapsize，也就是128K。映射完毕后会将地址空间的起始地址和大小保存在binder_state结构体中的mapped和mapsize变量中。

这里着重说一下open函数，它会调用Kernel Binder部分的binder_open函数，这部分源码位于内核源码中，这里展示的代码版本为goldfish3.4。

**用户态和内核态** 临时插入一个知识点:用户态和内核态 Intel的X86架构的CPU提供了0到3四个特权级，数字越小，权限越高，Linux操作系统中主要采用了0和3两个特权级，分别对应的就是内核态与用户态。用户态的特权级别低，因此进程在用户态下不经过系统调用是无法主动访问到内核空间中的数据的，这样用户无法随意的进入所有进程共享的内核空间，起到了保护的作用。下面来介绍下什么是用户态和内核态。 当一个进程在执行用户自己的代码时处于用户态，比如open函数，它运行在用户空间，当前的进程处于用户态。 当一个进程因为系统调用进入内核代码中执行时就处于内核态，比如open函数通过系统调用（__open()函数），查找到了open函数在Kernel Binder对应的函数为binder_open，这时binder_open运行在内核空间，当前的进程由用户态切换到内核态。

**kernel/goldfish/drivers/staging/android/binder.c**

```
static int binder_open(struct inode *nodp, struct file *filp)
{   //代表Binder进程
	struct binder_proc *proc;//1
	binder_debug(BINDER_DEBUG_OPEN_CLOSE, "binder_open: %d:%d\n",
		     current->group_leader->pid, current->pid);
    //分配内存空间
	proc = kzalloc(sizeof(*proc), GFP_KERNEL);//2
	if (proc == NULL)
		return -ENOMEM;
	get_task_struct(current);
	proc->tsk = current;
	INIT_LIST_HEAD(&proc->todo);
	init_waitqueue_head(&proc->wait);
	proc->default_priority = task_nice(current);
    //binder同步锁
	binder_lock(__func__);

	binder_stats_created(BINDER_STAT_PROC);
	hlist_add_head(&proc->proc_node, &binder_procs);
	proc->pid = current->group_leader->pid;
	INIT_LIST_HEAD(&proc->delivered_death);
	filp->private_data = proc;//3
    //binder同步锁释放
	binder_unlock(__func__);
	...
	return 0;
}
```

注释1处的binder_proc结构体代表binder进程，用于管理binder的各种信息。注释2处用于为binder_proc分配内存空间。注释3处将binder_proc赋值给file指针的private_data变量，后面的1.2小节会再次提到这个private_data变量。

#### **1.2 注册成为Binder机制的上下文管理者**

binder_become_context_manager函数用于将servicemanager注册成为Binder机制的上下文管理者，这个管理者在整个系统只有一个，代码如下所示。 **frameworks/native/cmds/servicemanager/binder.c**

```
int binder_become_context_manager(struct binder_state *bs)
{
    return ioctl(bs->fd, BINDER_SET_CONTEXT_MGR, 0);
}
```

ioctl函数会调用Binder驱动的binder_ioctl函数，binder_ioctl函数代码比较多，这里截取BINDER_SET_CONTEXT_MGR的处理部分，代码如下所示。 **kernel/goldfish/drivers/staging/android/binder.c**

```
static long binder_ioctl(struct file *filp, unsigned int cmd, unsigned long arg)
{
	int ret;
	struct binder_proc *proc = filp->private_data; //1
	struct binder_thread *thread;
	unsigned int size = _IOC_SIZE(cmd);
	void __user *ubuf = (void __user *)arg;
	trace_binder_ioctl(cmd, arg);

	ret = wait_event_interruptible(binder_user_error_wait, binder_stop_on_user_error < 2);
	if (ret)
		goto err_unlocked;

	binder_lock(__func__);
	thread = binder_get_thread(proc);//2
	if (thread == NULL) {
		ret = -ENOMEM;
		goto err;
	}

	switch (cmd) {
    ...
	case BINDER_SET_CONTEXT_MGR:
		if (binder_context_mgr_node != NULL) {//3
			printk(KERN_ERR "binder: BINDER_SET_CONTEXT_MGR already set\n");
			ret = -EBUSY;
			goto err;
		}
		ret = security_binder_set_context_mgr(proc->tsk);
		if (ret < 0)
			goto err;
		if (binder_context_mgr_uid != -1) {//4
			if (binder_context_mgr_uid != current->cred->euid) {//5
				printk(KERN_ERR "binder: BINDER_SET_"
				       "CONTEXT_MGR bad uid %d != %d\n",
				       current->cred->euid,
				       binder_context_mgr_uid);
				ret = -EPERM;
				goto err;
			}
		} else
			binder_context_mgr_uid = current->cred->euid;//6
		binder_context_mgr_node = binder_new_node(proc, NULL, NULL);//7
		if (binder_context_mgr_node == NULL) {
			ret = -ENOMEM;
			goto err;
		}
		binder_context_mgr_node->local_weak_refs++;
		binder_context_mgr_node->local_strong_refs++;
		binder_context_mgr_node->has_strong_ref = 1;
		binder_context_mgr_node->has_weak_ref = 1;
		break;
 ...
err_unlocked:
	trace_binder_ioctl_done(ret);
	return ret;
}
```

注释1处将file指针中的private_data变量赋值给binder_proc，这个private_data变量在binder_open函数中讲过，是一个binder_proc结构体。注释2处的binder_get_thread函数用于获取binder_thread，binder_thread结构体指的是binder线程，binder_get_thread函数内部会从传入的参数binder_proc中查找binder_thread，如果查询到直接返回，如果查询不到会创建一个新的binder_thread并返回。 注释3处的全局变量binder_context_mgr_node代表的是Binder机制的上下文管理者对应的一个Binder对象，如果它不为NULL，说明此前自身已经被注册为Binder的上下文管理者了，Binder的上下文管理者是不能重复注册的，因此会goto到err标签。 注释4处的全局变量binder_context_mgr_uid代表注册了Binder机制上下文管理者的进程的有效用户ID，如果它的值不为-1，说明此前已经有进程注册Binder的上下文管理者了，因此在注释5处判断当前进程的有效用户ID是否等于binder_context_mgr_uid，不等于就goto到err标签。 如果不满足注释4的条件，说明此前没有进程注册Binder机制的上下文管理者，就会在注释6处将当前进程的有效用户ID赋值给全局变量binder_context_mgr_uid，另外还会在注释7处调用binder_new_node函数创建一个Binder对象并赋值给全局变量binder_context_mgr_node。

#### **1.3 循环等待和处理client端发来的请求**

servicemanager成功注册成为Binder机制的上下文管理者后，servicemanager就是Binder机制的“总管”了，它需要在系统运行期间处理client端的请求，由于client端的请求不确定何时发送，因此需要通过无限循环来实现，实现这一需求的函数就是binder_loop。 **frameworks/native/cmds/servicemanager/binder.c**

```
void binder_loop(struct binder_state *bs, binder_handler func)
{
    int res;
    struct binder_write_read bwr;
    uint32_t readbuf[32];

    bwr.write_size = 0;
    bwr.write_consumed = 0;
    bwr.write_buffer = 0;

    readbuf[0] = BC_ENTER_LOOPER;
    binder_write(bs, readbuf, sizeof(uint32_t));//1

    for (;;) {
        bwr.read_size = sizeof(readbuf);
        bwr.read_consumed = 0;
        bwr.read_buffer = (uintptr_t) readbuf;

        res = ioctl(bs->fd, BINDER_WRITE_READ, &bwr);//2

        if (res < 0) {
            ALOGE("binder_loop: ioctl failed (%s)\n", strerror(errno));
            break;
        }

        res = binder_parse(bs, 0, (uintptr_t) readbuf, bwr.read_consumed, func);//3
        if (res == 0) {
            ALOGE("binder_loop: unexpected reply?!\n");
            break;
        }
        if (res < 0) {
            ALOGE("binder_loop: io error %d %s\n", res, strerror(errno));
            break;
        }
    }
}
```

注释1处将BC_ENTER_LOOPER指令通过binder_write函数写入到Binder驱动中，这样当前线程（ServiceManager的主线程）就成为了一个Binder线程，这样就可以处理进程间的请求了。 在无限循环中不断的调用注释2处的ioctl函数，它不断的使用BINDER_WRITE_READ指令查询Binder驱动中是否有新的请求，如果有就交给注释3处的binder_parse函数处理。如果没有，当前线程就会在Binder驱动中睡眠，等待新的进程间请求。

由于binder_write函数的调用链中涉及到了内核空间和用户空间的交互，因此这里着重讲解下。

**frameworks/native/cmds/servicemanager/binder.c**

```
int binder_write(struct binder_state *bs, void *data, size_t len)
{
    struct binder_write_read bwr;//1
    int res;

    bwr.write_size = len;
    bwr.write_consumed = 0;
    bwr.write_buffer = (uintptr_t) data;//2
    bwr.read_size = 0;
    bwr.read_consumed = 0;
    bwr.read_buffer = 0;
    res = ioctl(bs->fd, BINDER_WRITE_READ, &bwr);//3
    if (res < 0) {
        fprintf(stderr,"binder_write: ioctl failed (%s)\n",
                strerror(errno));
    }
    return res;
}
```

注释1处定义binder_write_read结构体，接下来的代码对bwr进行赋值，其中需要注意的是，注释2处的data的值为BC_ENTER_LOOPER。注释3处的ioctl函数将会bwr中的数据发送给binder驱动，我们已经知道了ioctl函数在Kernel Binder中对应的函数为binder_ioctl，此前分析过这个函数，这里截取BINDER_WRITE_READ命令处理部分。

**kernel/goldfish/drivers/staging/android/binder.c**

```
static long binder_ioctl(struct file *filp, unsigned int cmd, unsigned long arg)
{   
    ...
    void __user *ubuf = (void __user *)arg;
    ...
	switch (cmd) {
	case BINDER_WRITE_READ: {
		struct binder_write_read bwr;
		if (size != sizeof(struct binder_write_read)) {
			ret = -EINVAL;
			goto err;
		}
		if (copy_from_user(&bwr, ubuf, sizeof(bwr))) {//1
			ret = -EFAULT;
			goto err;
		}
		binder_debug(BINDER_DEBUG_READ_WRITE,
			     "binder: %d:%d write %ld at %08lx, read %ld at %08lx\n",
			     proc->pid, thread->pid, bwr.write_size, bwr.write_buffer,
			     bwr.read_size, bwr.read_buffer);

		if (bwr.write_size > 0) {//2
			ret = binder_thread_write(proc, thread, (void __user *)bwr.write_buffer, bwr.write_size, &bwr.write_consumed);//3
			trace_binder_write_done(ret);
			if (ret < 0) {
				bwr.read_consumed = 0;
				if (copy_to_user(ubuf, &bwr, sizeof(bwr)))
					ret = -EFAULT;
				goto err;
			}
		}
	    ...
		binder_debug(BINDER_DEBUG_READ_WRITE,
			     "binder: %d:%d wrote %ld of %ld, read return %ld of %ld\n",
			     proc->pid, thread->pid, bwr.write_consumed, bwr.write_size,
			     bwr.read_consumed, bwr.read_size);
		if (copy_to_user(ubuf, &bwr, sizeof(bwr))) {//4
			ret = -EFAULT;
			goto err;
		}
		break;
	}
   ...
	return ret;
}
```

注释1处的copy_from_user函数，在本系列的第一篇文章[Android Binder原理（一）学习Binder前必须要了解的知识点](http://liuwangshu.cn/framework/binder/1-introt.html)提过。在这里，它用于将把用户空间数据ubuf拷贝出来保存到内核数据bwr（binder_write_read结构体）中。 注释2处，bwr的输入缓存区有数据时，会调用注释3处的binder_thread_write函数来处理BC_ENTER_LOOPER协议，其内部会将目标线程的状态设置为BINDER_LOOPER_STATE_ENTERED，这样目标线程就是一个Binder线程。 注释4处通过copy_to_user函数将内核空间数据bwr拷贝到用户空间。

### **2.总结**

ServiceManager的启动过程实际上就是分析ServiceManager的入口函数，在入口函数中主要做了三件事，本篇文章深入到内核源码来对这三件逐一进行分析，由于涉及的函数比较多，这篇文章只介绍了我们需要掌握的，剩余大家可以自行阅读源码，比如binder_thread_write、copy_to_user函数。




链接：https://juejin.cn/post/6844904000840531975