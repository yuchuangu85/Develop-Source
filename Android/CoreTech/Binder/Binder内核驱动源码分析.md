<h1 align="center">Binder内核驱动源码分析</h1>

[toc]

## 数据结构定义

### ioctl定义

```c
// 在binder文件被打开后，其私有数据（private_data）的类型：struct binder_proc
// 在这个数据结构中，主要包含了当前进程、进程ID、内存映射信息、Binder的统计信息和线程信息等。
// 在用户空间对Binder驱动程序进行控制主要使用的接口是mmap、poll和ioctl，ioctl主要使用的ID为：
     
#define BINDER_WRITE_READ        _IOWR('b', 1, struct binder_write_read)
#define BINDER_SET_IDLE_TIMEOUT  _IOW('b', 3, int64_t)
#define BINDER_SET_MAX_THREADS   _IOW('b', 5, size_t)
#define BINDER_SET_IDLE_PRIORITY _IOW('b', 6, int)
#define BINDER_SET_CONTEXT_MGR   _IOW('b', 7, int)
#define BINDER_THREAD_EXIT       _IOW('b', 8, int)
#define BINDER_VERSION           _IOWR('b', 9, struct binder_version)
 
// BR_XXX等宏为BinderDriverReturnProtocol，表示Binder驱动返回协议。
// BC_XXX等宏为BinderDriverCommandProtocol，表示Binder驱动命令协议。
```

### binder_write_read

```c
// BINDER_WRITE_READ是最重要的ioctl，它使用一个数据结构binder_write_read定义读写的数据。
// 该数据结构能够很好的处理数据传输的同步和异步情形。
// 当write_size为0时，只处理读操作，当read_size为0时只处理写操作，
// 二者都不为0时，先执行写操作再执行读操作，以此实现同步。
struct binder_write_read {
     signed long write_size;
     signed long write_consumed;
     unsigned long write_buffer;
     signed long read_size;
     signed long read_consumed;
     unsigned long read_buffer;
};
```

### binder_transaction_data

```c
struct binder_transaction_data {
    /* The first two are only used for bcTRANSACTION and brTRANSACTION,
     * identifying the target and contents of the transaction.
     */
    union {
        size_t  handle;     /* target descriptor of command transaction */
        void    *ptr;       /* target descriptor of return transaction */
    } target;
    void        *cookie;    /* target object cookie */
    unsigned int    code;   /* transaction command */
 
    /* General information about the transaction. */
    unsigned int    flags;
    pid_t       sender_pid;
    uid_t       sender_euid;
    size_t      data_size;  /* number of bytes of data */
    size_t      offsets_size;   /* number of bytes of offsets */
 
    /* If this transaction is inline, the data immediately
     * follows here; otherwise, it ends with a pointer to
     * the data buffer.
     */
    union {
        struct {
            /* transaction data */
            const void  *buffer;
            /* offsets from buffer to flat_binder_object structs */
            const void  *offsets;
        } ptr;
        uint8_t buf[8];
    } data;
};
```

### binder_fops

```c
static struct file_operations binder_fops = {  
    .owner = THIS_MODULE,  
    .poll = binder_poll,  
    .unlocked_ioctl = binder_ioctl,  
    .mmap = binder_mmap,  
    .open = binder_open,  
    .flush = binder_flush,  
    .release = binder_release,  
}; 
```

### miscdevice

```c
static struct miscdevice binder_miscdev = {
    .minor = MISC_DYNAMIC_MINOR,
    .name = "binder",
    .fops = &binder_fops
};
```

### binder_proc

```c
struct binder_proc {
    struct hlist_node proc_node;
    struct rb_root threads;                 // 线程红黑树，用于保存所有用户进程中的线程
    struct rb_root nodes;                   // Binder实体红黑树
    struct rb_root refs_by_desc;            // Binder引用红黑树，用句柄作为Key
    struct rb_root refs_by_node;            // Binder引用红黑树，用实体节点的地址作为Key
    int pid;                                // 进程ID
    struct vm_area_struct *vma;             // 用户空间的虚拟地址数据结构
    struct task_struct *tsk;
    struct files_struct *files;
    struct hlist_node deferred_work_node;
    int deferred_work;
    void *buffer;                           // 物理内存在内核空间中的地址
    ptrdiff_t user_buffer_offset;           // 内核使用的虚拟地址与进程使用的虚拟地址之间的差值
                                            // user_buffer_offset = proc_start - kernel_start;
 
    struct list_head buffers;
    struct rb_root free_buffers;            // 空闲的地址区域
    struct rb_root allocated_buffers;       // 正在使用的地址区域
    size_t free_async_space;
 
    struct page **pages;                    // 物理页面的数据结构
    size_t buffer_size;                     // 映射的内存大小
    uint32_t buffer_free;
    struct list_head todo;
    wait_queue_head_t wait;
    struct binder_stats stats;
    struct list_head delivered_death;
    int max_threads;                        // 用户请求最大线程数
    int requested_threads;
    int requested_threads_started;
    int ready_threads;
    long default_priority;
};
```

### binder_thread

```c
struct binder_thread {
    struct binder_proc *proc;               // 该线程相关的进程
    struct rb_node rb_node;                 // 进程中的所有线程红黑树
    int pid;
    int looper;
    struct binder_transaction *transaction_stack;
    struct list_head todo;
    uint32_t return_error;
    uint32_t return_error2;
    wait_queue_head_t wait;
    struct binder_stats stats;
};
```

### binder_node

```c
// 表示一个Binder实体
struct binder_node {
    int debug_id;
    struct binder_work work;
    union {
        struct rb_node rb_node;
        struct hlist_node dead_node;
    };
    struct binder_proc *proc;
    struct hlist_head refs;
    int internal_strong_refs;
    int local_weak_refs;
    int local_strong_refs;
    void __user *ptr;
    void __user *cookie;
    unsigned has_strong_ref : 1;
    unsigned pending_strong_ref : 1;
    unsigned has_weak_ref : 1;
    unsigned pending_weak_ref : 1;
    unsigned has_async_transaction : 1;
    unsigned accept_fds : 1;
    int min_priority : 8;
    struct list_head async_todo;
};
```

### binder_ref

```c
struct binder_ref {
    /* Lookups needed: */
    /*   node + proc => ref (transaction) */
    /*   desc + proc => ref (transaction, inc/dec ref) */
    /*   node => refs + procs (proc exit) */
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

### binder_ref_death

```c
struct binder_ref_death {
    struct binder_work work;
    void __user *cookie;
};
```

### binder_buffer

```c
struct binder_buffer {
    struct list_head entry;                 // 链表头
    struct rb_node rb_node;                 // 缓冲空间节点
     
    unsigned free : 1;                      // 当前区域为空闲
    unsigned allow_user_free : 1;
    unsigned async_transaction : 1;
    unsigned debug_id : 29;
 
    struct binder_transaction *transaction;
 
    struct binder_node *target_node;
    size_t data_size;
    size_t offsets_size;
    uint8_t data[0];
};
```

## 函数定义

### binder_init

```c
static int __init binder_init(void)
{
    int ret;
 
    binder_proc_dir_entry_root = proc_mkdir("binder", NULL);
    if (binder_proc_dir_entry_root)
        binder_proc_dir_entry_proc = proc_mkdir("proc", binder_proc_dir_entry_root);
    ret = misc_register(&binder_miscdev);
    if (binder_proc_dir_entry_root) {
        create_proc_read_entry("state", S_IRUGO, binder_proc_dir_entry_root, 
                               binder_read_proc_state, NULL);
        create_proc_read_entry("stats", S_IRUGO, binder_proc_dir_entry_root, 
                               binder_read_proc_stats, NULL);
        create_proc_read_entry("transactions", S_IRUGO, binder_proc_dir_entry_root, 
                               binder_read_proc_transactions, NULL);
        create_proc_read_entry("transaction_log", S_IRUGO, binder_proc_dir_entry_root, 
                               binder_read_proc_transaction_log, &binder_transaction_log);
        create_proc_read_entry("failed_transaction_log", S_IRUGO, binder_proc_dir_entry_root, 
                               binder_read_proc_transaction_log, &binder_transaction_log_failed);
    }
    return ret;
}
```

### binder_open

```c
// 这个函数的主要作用是创建一个struct binder_proc数据结构来保存每个打开设备文件/dev/binder的进程的上下文信息，
// 并且将这个进程上下文信息保存在打开文件结构struct file的私有数据成员变量private_data中。这样，在执行文件操作时，
// 就通过打开文件结构struct file来取回这个进程上下文信息。这个进程上下文信息同时还会保存在一个全局哈希表binder_procs中，
// 供驱动程序内部使用。
static int binder_open(struct inode *nodp, struct file *filp)
{
    struct binder_proc *proc;
 
    if (binder_debug_mask & BINDER_DEBUG_OPEN_CLOSE)
        printk(KERN_INFO "binder_open: %d:%d\n", current->group_leader->pid, current->pid);
 
    proc = kzalloc(sizeof(*proc), GFP_KERNEL);
    if (proc == NULL)
        return -ENOMEM;
         
    get_task_struct(current);
    proc->tsk = current;
    INIT_LIST_HEAD(&proc->todo);
    init_waitqueue_head(&proc->wait);
    proc->default_priority = task_nice(current);
    mutex_lock(&binder_lock);
    binder_stats.obj_created[BINDER_STAT_PROC]++;
    hlist_add_head(&proc->proc_node, &binder_procs);
    proc->pid = current->group_leader->pid;
    INIT_LIST_HEAD(&proc->delivered_death);
    filp->private_data = proc;
    mutex_unlock(&binder_lock);
 
    if (binder_proc_dir_entry_proc) {
        char strbuf[11];
        snprintf(strbuf, sizeof(strbuf), "%u", proc->pid);
        remove_proc_entry(strbuf, binder_proc_dir_entry_proc);
        create_proc_read_entry(strbuf, S_IRUGO, binder_proc_dir_entry_proc, binder_read_proc_proc, proc);
    }
 
    return 0;
}
```

### binder_mmap

```c
static int binder_mmap(struct file *filp, struct vm_area_struct *vma)
{
    int ret;
    struct vm_struct *area;
    struct binder_proc *proc = filp->private_data;
    const char *failure_string;
    struct binder_buffer *buffer;
 
    // 检查映射的内存大小不能超过4M
    if ((vma->vm_end - vma->vm_start) > SZ_4M)
        vma->vm_end = vma->vm_start + SZ_4M;
 
    if (binder_debug_mask & BINDER_DEBUG_OPEN_CLOSE)
        printk(KERN_INFO
            "binder_mmap: %d %lx-%lx (%ld K) vma %lx pagep %lx\n",
            proc->pid, vma->vm_start, vma->vm_end,
            (vma->vm_end - vma->vm_start) / SZ_1K, vma->vm_flags,
            (unsigned long)pgprot_val(vma->vm_page_prot));
 
    if (vma->vm_flags & FORBIDDEN_MMAP_FLAGS) {
        ret = -EPERM;
        failure_string = "bad vm_flags";
        goto err_bad_arg;
    }
    vma->vm_flags = (vma->vm_flags | VM_DONTCOPY) & ~VM_MAYWRITE;
 
    if (proc->buffer) {
        ret = -EBUSY;
        failure_string = "already mapped";
        goto err_already_mapped;
    }
 
    // 获取内核空间的虚拟地址空间，和用户进程的大小相同
    area = get_vm_area(vma->vm_end - vma->vm_start, VM_IOREMAP);
    if (area == NULL) {
        ret = -ENOMEM;
        failure_string = "get_vm_area";
        goto err_get_vm_area_failed;
    }
     
    // 将内核空间的起始地址存入binder_proc结构的buffer变量中，
    // 并计算内核空间地址与用户进程空间地址的偏移
    proc->buffer = area->addr;
    proc->user_buffer_offset = vma->vm_start - (uintptr_t)proc->buffer;
 
#ifdef CONFIG_CPU_CACHE_VIPT
    if (cache_is_vipt_aliasing()) {
        while (CACHE_COLOUR((vma->vm_start ^ (uint32_t)proc->buffer))) {
            printk(KERN_INFO "binder_mmap: %d %lx-%lx maps %p bad alignment\n", proc->pid, 
                   vma->vm_start, vma->vm_end, proc->buffer);
            vma->vm_start += PAGE_SIZE;
        }
    }
#endif
 
    // 申请物理页面，并计算虚拟地址空间大小
    proc->pages = kzalloc(sizeof(proc->pages[0]) * ((vma->vm_end - vma->vm_start) / PAGE_SIZE), GFP_KERNEL);
    if (proc->pages == NULL) {
        ret = -ENOMEM;
        failure_string = "alloc page array";
        goto err_alloc_pages_failed;
    }
    proc->buffer_size = vma->vm_end - vma->vm_start;
 
    vma->vm_ops = &binder_vm_ops;
    vma->vm_private_data = proc;
 
    // 将物理页面映射到用户空间和内核空间
    // Server端的用户空间虚拟地址与内核端的虚拟地址共享同一物理内存区域
    if (binder_update_page_range(proc, 1, proc->buffer, proc->buffer + PAGE_SIZE, vma)) {
        ret = -ENOMEM;
        failure_string = "alloc small buf";
        goto err_alloc_small_buf_failed;
    }
     
    // 将内核空间的地址区域划分成若干binder_buffer数据结构进行分配，并加入到链表结构中
    buffer = proc->buffer;
    INIT_LIST_HEAD(&proc->buffers);
    list_add(&buffer->entry, &proc->buffers);
    buffer->free = 1;
    binder_insert_free_buffer(proc, buffer);
    proc->free_async_space = proc->buffer_size / 2;
    barrier();
    proc->files = get_files_struct(current);
    proc->vma = vma;
 
    /*printk(KERN_INFO "binder_mmap: %d %lx-%lx maps %p\n", proc->pid, 
      vma->vm_start, vma->vm_end, proc->buffer);*/
    return 0;
 
err_alloc_small_buf_failed:
    kfree(proc->pages);
    proc->pages = NULL;
err_alloc_pages_failed:
    vfree(proc->buffer);
    proc->buffer = NULL;
err_get_vm_area_failed:
err_already_mapped:
err_bad_arg:
    printk(KERN_ERR "binder_mmap: %d %lx-%lx %s failed %d\n", proc->pid, vma->vm_start, 
           vma->vm_end, failure_string, ret);
    return ret;
}
```

### binder_update_page_range

```c
// 将物理页面映射到用户空间和内核空间
static int binder_update_page_range(struct binder_proc *proc, int allocate,
    void *start, void *end, struct vm_area_struct *vma)
{
    void *page_addr;
    unsigned long user_page_addr;
    struct vm_struct tmp_area;
    struct page **page;
    struct mm_struct *mm;
 
    if (binder_debug_mask & BINDER_DEBUG_BUFFER_ALLOC)
        printk(KERN_INFO "binder: %d: %s pages %p-%p\n",
               proc->pid, allocate ? "allocate" : "free", start, end);
 
    if (end <= start)
        return 0;
 
    if (vma)
        mm = NULL;
    else
        mm = get_task_mm(proc->tsk);
 
    if (mm) {
        down_write(&mm->mmap_sem);
        vma = proc->vma;
    }
 
    // 如果表示为0，则跳转到释放物理内存的代码段
    if (allocate == 0)
        goto free_range;
 
    if (vma == NULL) {
        printk(KERN_ERR "binder: %d: binder_alloc_buf failed to "
               "map pages in userspace, no vma\n", proc->pid);
        goto err_no_vma;
    }
 
    // 分配物理内存，并映射到用户虚拟地址空间和内核虚拟地址空间
    for (page_addr = start; page_addr < end; page_addr += PAGE_SIZE) {
        int ret;
        struct page **page_array_ptr;
        page = &proc->pages[(page_addr - proc->buffer) / PAGE_SIZE];
 
        BUG_ON(*page);
        // alloc_page分配内存
        *page = alloc_page(GFP_KERNEL | __GFP_ZERO);
        if (*page == NULL) {
            printk(KERN_ERR "binder: %d: binder_alloc_buf failed "
                   "for page at %p\n", proc->pid, page_addr);
            goto err_alloc_page_failed;
        }
        tmp_area.addr = page_addr;
        tmp_area.size = PAGE_SIZE + PAGE_SIZE /* guard page? */;
        page_array_ptr = page;
         
        // 映射内核虚拟地址空间
        ret = map_vm_area(&tmp_area, PAGE_KERNEL, &page_array_ptr);
        if (ret) {
            printk(KERN_ERR "binder: %d: binder_alloc_buf failed "
                   "to map page at %p in kernel\n",
                   proc->pid, page_addr);
            goto err_map_kernel_failed;
        }
         
        // 根据偏移查找用户虚拟地址
        user_page_addr = (uintptr_t)page_addr + proc->user_buffer_offset;
        // 映射到用户虚拟地址空间
        ret = vm_insert_page(vma, user_page_addr, page[0]);
        if (ret) {
            printk(KERN_ERR "binder: %d: binder_alloc_buf failed "
                   "to map page at %lx in userspace\n",
                   proc->pid, user_page_addr);
            goto err_vm_insert_page_failed;
        }
        /* vm_insert_page does not seem to increment the refcount */
    }
    if (mm) {
        up_write(&mm->mmap_sem);
        mmput(mm);
    }
    return 0;
 
// 释放物理内存
free_range:
    for (page_addr = end - PAGE_SIZE; page_addr >= start;
         page_addr -= PAGE_SIZE) {
        page = &proc->pages[(page_addr - proc->buffer) / PAGE_SIZE];
        if (vma)
            zap_page_range(vma, (uintptr_t)page_addr +
                proc->user_buffer_offset, PAGE_SIZE, NULL);
err_vm_insert_page_failed:
        unmap_kernel_range((unsigned long)page_addr, PAGE_SIZE);
err_map_kernel_failed:
        __free_page(*page);
        *page = NULL;
err_alloc_page_failed:
        ;
    }
err_no_vma:
    if (mm) {
        up_write(&mm->mmap_sem);
        mmput(mm);
    }
    return -ENOMEM;
}
```

### binder_ioctl

```c
static long binder_ioctl(struct file *filp, unsigned int cmd, unsigned long arg) {
    int ret;
    struct binder_proc *proc = filp->private_data;
    struct binder_thread *thread;
    unsigned int size = _IOC_SIZE(cmd);
    void __user *ubuf = (void __user *)arg;
 
    ret = wait_event_interruptible(binder_user_error_wait, binder_stop_on_user_error < 2);
    if (ret)
        return ret;
 
    mutex_lock(&binder_lock);
    // 从相关进程中获取binder_thread数据结构信息
    thread = binder_get_thread(proc);
    if (thread == NULL) {
        ret = -ENOMEM;
        goto err;
    }
 
    switch (cmd) {
    case BINDER_SET_CONTEXT_MGR:
        if (binder_context_mgr_node != NULL) {
            printk(KERN_ERR "binder: BINDER_SET_CONTEXT_MGR already set\n");
            ret = -EBUSY;
            goto err;
        }
         
        if (binder_context_mgr_uid != -1) {
            if (binder_context_mgr_uid != current->cred->euid) {
                printk(KERN_ERR "binder: BINDER_SET_"
                    "CONTEXT_MGR bad uid %d != %d\n",
                    current->cred->euid,
                    binder_context_mgr_uid);
                ret = -EPERM;
                goto err;
            }
        } else {
            // 第一次进入时初始化ServiceManager的uid
            binder_context_mgr_uid = current->cred->euid;
        }
         
        // 创建ServiceManager的Binder实体
        binder_context_mgr_node = binder_new_node(proc, NULL, NULL);
        if (binder_context_mgr_node == NULL) {
            ret = -ENOMEM;
            goto err;
        }
         
        // 初始化ServiceManager的Binder实体
        binder_context_mgr_node->local_weak_refs++;
        binder_context_mgr_node->local_strong_refs++;
        binder_context_mgr_node->has_strong_ref = 1;
        binder_context_mgr_node->has_weak_ref = 1;
        break;
         
    case BINDER_WRITE_READ: {
        struct binder_write_read bwr;
        if (size != sizeof(struct binder_write_read)) {
            ret = -EINVAL;
            goto err;
        }
         
        // 从用户空间拷贝数据到内核空间
        if (copy_from_user(&bwr, ubuf, sizeof(bwr))) {
            ret = -EFAULT;
            goto err;
        }
        if (binder_debug_mask & BINDER_DEBUG_READ_WRITE)
            printk(KERN_INFO "binder: %d:%d write %ld at %08lx, read %ld at %08lx\n",
            proc->pid, thread->pid, bwr.write_size, bwr.write_buffer, bwr.read_size, bwr.read_buffer);
             
        // 如果有写入数据，则调用binder_thread_write
        if (bwr.write_size > 0) {
            ret = binder_thread_write(proc, thread, (void __user *)bwr.write_buffer, bwr.write_size, 
                                      &bwr.write_consumed);
            if (ret < 0) {
                bwr.read_consumed = 0;
                if (copy_to_user(ubuf, &bwr, sizeof(bwr)))
                    ret = -EFAULT;
                goto err;
            }
        }
         
        // 如果有读入数据，则调用binder_thread_read
        if (bwr.read_size > 0) {
            ret = binder_thread_read(proc, thread, (void __user *)bwr.read_buffer, bwr.read_size, 
                                     &bwr.read_consumed, filp->f_flags & O_NONBLOCK);
            if (!list_empty(&proc->todo))
                wake_up_interruptible(&proc->wait);
            if (ret < 0) {
                if (copy_to_user(ubuf, &bwr, sizeof(bwr)))
                    ret = -EFAULT;
                goto err;
            }
        }
        if (binder_debug_mask & BINDER_DEBUG_READ_WRITE)
            printk(KERN_INFO "binder: %d:%d wrote %ld of %ld, read return %ld of %ld\n",
            proc->pid, thread->pid, bwr.write_consumed, bwr.write_size, bwr.read_consumed, bwr.read_size);
             
        // 从内核空间拷贝数据到用户空间
        if (copy_to_user(ubuf, &bwr, sizeof(bwr))) {
            ret = -EFAULT;
            goto err;
        }
        break;
    }
     
    case BINDER_SET_MAX_THREADS:
        if (copy_from_user(&proc->max_threads, ubuf, sizeof(proc->max_threads))) {
            ret = -EINVAL;
            goto err;
        }
        break;
         
    case BINDER_THREAD_EXIT:
        if (binder_debug_mask & BINDER_DEBUG_THREADS)
            printk(KERN_INFO "binder: %d:%d exit\n", proc->pid, thread->pid);
        binder_free_thread(proc, thread);
        thread = NULL;
        break;
         
    case BINDER_VERSION:
        if (size != sizeof(struct binder_version)) {
            ret = -EINVAL;
            goto err;
        }
        if (put_user(BINDER_CURRENT_PROTOCOL_VERSION, 
                     &((struct binder_version *)ubuf)->protocol_version)) {
            ret = -EINVAL;
            goto err;
        }
        break;
     
    default:
        ret = -EINVAL;
        goto err;
    }
    ret = 0;
err:
    if (thread)
        thread->looper &= ~BINDER_LOOPER_STATE_NEED_RETURN;
    mutex_unlock(&binder_lock);
    wait_event_interruptible(binder_user_error_wait, binder_stop_on_user_error < 2);
    if (ret && ret != -ERESTARTSYS)
        printk(KERN_INFO "binder: %d:%d ioctl %x %lx returned %d\n", 
               proc->pid, current->pid, cmd, arg, ret);
    return ret;
}
```

### binder_get_thread

```c
static struct binder_thread *binder_get_thread(struct binder_proc *proc) {
    struct binder_thread *thread = NULL;
    struct rb_node *parent = NULL;
    struct rb_node **p = &proc->threads.rb_node;
 
    // 根据当前线程的PID查找红黑树节点
    while (*p) {
        parent = *p;
        thread = rb_entry(parent, struct binder_thread, rb_node);
 
        if (current->pid < thread->pid)
            p = &(*p)->rb_left;
        else if (current->pid > thread->pid)
            p = &(*p)->rb_right;
        else
            break;
    }
     
    // 如果查找到对应的节点，则填充binder_thread数据结构，并返回
    if (*p == NULL) {
        thread = kzalloc(sizeof(*thread), GFP_KERNEL);
        if (thread == NULL)
            return NULL;
        binder_stats.obj_created[BINDER_STAT_THREAD]++;
        thread->proc = proc;
        thread->pid = current->pid;
        init_waitqueue_head(&thread->wait);
        INIT_LIST_HEAD(&thread->todo);
        rb_link_node(&thread->rb_node, parent, p);
        rb_insert_color(&thread->rb_node, &proc->threads);
        thread->looper |= BINDER_LOOPER_STATE_NEED_RETURN;
        thread->return_error = BR_OK;
        thread->return_error2 = BR_OK;
    }
     
    return thread;
}
```

### binder_new_node

```c
// 创建新的Binder实体
static struct binder_node *
binder_new_node(struct binder_proc *proc, void __user *ptr, void __user *cookie)
{
    struct rb_node **p = &proc->nodes.rb_node;
    struct rb_node *parent = NULL;
    struct binder_node *node;
 
    while (*p) {
        parent = *p;
        node = rb_entry(parent, struct binder_node, rb_node);
 
        if (ptr < node->ptr)
            p = &(*p)->rb_left;
        else if (ptr > node->ptr)
            p = &(*p)->rb_right;
        else
            return NULL;
    }
 
    node = kzalloc(sizeof(*node), GFP_KERNEL);
    if (node == NULL)
        return NULL;
    binder_stats.obj_created[BINDER_STAT_NODE]++;
    rb_link_node(&node->rb_node, parent, p);
    rb_insert_color(&node->rb_node, &proc->nodes);
    node->debug_id = ++binder_last_id;
    node->proc = proc;
    node->ptr = ptr;
    node->cookie = cookie;
    node->work.type = BINDER_WORK_NODE;
    INIT_LIST_HEAD(&node->work.entry);
    INIT_LIST_HEAD(&node->async_todo);
    if (binder_debug_mask & BINDER_DEBUG_INTERNAL_REFS)
        printk(KERN_INFO "binder: %d:%d node %d u%p c%p created\n",
               proc->pid, current->pid, node->debug_id,
               node->ptr, node->cookie);
    return node;
}
```

### binder_thread_write

```c
int
binder_thread_write(struct binder_proc *proc, struct binder_thread *thread,
            void __user *buffer, int size, signed long *consumed)
{
    uint32_t cmd;
    void __user *ptr = buffer + *consumed;
    void __user *end = buffer + size;
 
    while (ptr < end && thread->return_error == BR_OK) {
        if (get_user(cmd, (uint32_t __user *)ptr))
            return -EFAULT;
        ptr += sizeof(uint32_t);
        if (_IOC_NR(cmd) < ARRAY_SIZE(binder_stats.bc)) {
            binder_stats.bc[_IOC_NR(cmd)]++;
            proc->stats.bc[_IOC_NR(cmd)]++;
            thread->stats.bc[_IOC_NR(cmd)]++;
        }
        switch (cmd) {
        case BC_INCREFS:
        case BC_ACQUIRE:
        case BC_RELEASE:
        case BC_DECREFS: {
            uint32_t target;
            struct binder_ref *ref;
            const char *debug_string;
 
            if (get_user(target, (uint32_t __user *)ptr))
                return -EFAULT;
            ptr += sizeof(uint32_t);
            if (target == 0 && binder_context_mgr_node &&
                (cmd == BC_INCREFS || cmd == BC_ACQUIRE)) {
                ref = binder_get_ref_for_node(proc,
                           binder_context_mgr_node);
                if (ref->desc != target) {
                    binder_user_error("binder: %d:"
                        "%d tried to acquire "
                        "reference to desc 0, "
                        "got %d instead\n",
                        proc->pid, thread->pid,
                        ref->desc);
                }
            } else
                ref = binder_get_ref(proc, target);
            if (ref == NULL) {
                binder_user_error("binder: %d:%d refcou"
                    "nt change on invalid ref %d\n",
                    proc->pid, thread->pid, target);
                break;
            }
            switch (cmd) {
            case BC_INCREFS:
                debug_string = "IncRefs";
                binder_inc_ref(ref, 0, NULL);
                break;
            case BC_ACQUIRE:
                debug_string = "Acquire";
                binder_inc_ref(ref, 1, NULL);
                break;
            case BC_RELEASE:
                debug_string = "Release";
                binder_dec_ref(ref, 1);
                break;
            case BC_DECREFS:
            default:
                debug_string = "DecRefs";
                binder_dec_ref(ref, 0);
                break;
            }
            if (binder_debug_mask & BINDER_DEBUG_USER_REFS)
                printk(KERN_INFO "binder: %d:%d %s ref %d desc %d s %d w %d for node %d\n",
                       proc->pid, thread->pid, debug_string, ref->debug_id, ref->desc, 
                       ref->strong, ref->weak, ref->node->debug_id);
            break;
        }
        case BC_INCREFS_DONE:
        case BC_ACQUIRE_DONE: {
            void __user *node_ptr;
            void *cookie;
            struct binder_node *node;
 
            if (get_user(node_ptr, (void * __user *)ptr))
                return -EFAULT;
            ptr += sizeof(void *);
            if (get_user(cookie, (void * __user *)ptr))
                return -EFAULT;
            ptr += sizeof(void *);
            node = binder_get_node(proc, node_ptr);
            if (node == NULL) {
                binder_user_error("binder: %d:%d "
                    "%s u%p no match\n",
                    proc->pid, thread->pid,
                    cmd == BC_INCREFS_DONE ?
                    "BC_INCREFS_DONE" :
                    "BC_ACQUIRE_DONE",
                    node_ptr);
                break;
            }
            if (cookie != node->cookie) {
                binder_user_error("binder: %d:%d %s u%p node %d"
                    " cookie mismatch %p != %p\n",
                    proc->pid, thread->pid,
                    cmd == BC_INCREFS_DONE ?
                    "BC_INCREFS_DONE" : "BC_ACQUIRE_DONE",
                    node_ptr, node->debug_id,
                    cookie, node->cookie);
                break;
            }
            if (cmd == BC_ACQUIRE_DONE) {
                if (node->pending_strong_ref == 0) {
                    binder_user_error("binder: %d:%d "
                        "BC_ACQUIRE_DONE node %d has "
                        "no pending acquire request\n",
                        proc->pid, thread->pid,
                        node->debug_id);
                    break;
                }
                node->pending_strong_ref = 0;
            } else {
                if (node->pending_weak_ref == 0) {
                    binder_user_error("binder: %d:%d "
                        "BC_INCREFS_DONE node %d has "
                        "no pending increfs request\n",
                        proc->pid, thread->pid,
                        node->debug_id);
                    break;
                }
                node->pending_weak_ref = 0;
            }
            binder_dec_node(node, cmd == BC_ACQUIRE_DONE, 0);
            if (binder_debug_mask & BINDER_DEBUG_USER_REFS)
                printk(KERN_INFO "binder: %d:%d %s node %d ls %d lw %d\n",
                       proc->pid, thread->pid, cmd == BC_INCREFS_DONE ? "BC_INCREFS_DONE" : "BC_ACQUIRE_DONE", 
                       node->debug_id, node->local_strong_refs, node->local_weak_refs);
            break;
        }
        case BC_ATTEMPT_ACQUIRE:
            printk(KERN_ERR "binder: BC_ATTEMPT_ACQUIRE not supported\n");
            return -EINVAL;
        case BC_ACQUIRE_RESULT:
            printk(KERN_ERR "binder: BC_ACQUIRE_RESULT not supported\n");
            return -EINVAL;
 
        case BC_FREE_BUFFER: {
            void __user *data_ptr;
            struct binder_buffer *buffer;
 
            if (get_user(data_ptr, (void * __user *)ptr))
                return -EFAULT;
            ptr += sizeof(void *);
 
            buffer = binder_buffer_lookup(proc, data_ptr);
            if (buffer == NULL) {
                binder_user_error("binder: %d:%d "
                    "BC_FREE_BUFFER u%p no match\n",
                    proc->pid, thread->pid, data_ptr);
                break;
            }
            if (!buffer->allow_user_free) {
                binder_user_error("binder: %d:%d "
                    "BC_FREE_BUFFER u%p matched "
                    "unreturned buffer\n",
                    proc->pid, thread->pid, data_ptr);
                break;
            }
            if (binder_debug_mask & BINDER_DEBUG_FREE_BUFFER)
                printk(KERN_INFO "binder: %d:%d BC_FREE_BUFFER u%p found buffer %d for %s transaction\n",
                       proc->pid, thread->pid, data_ptr, buffer->debug_id,
                       buffer->transaction ? "active" : "finished");
 
            if (buffer->transaction) {
                buffer->transaction->buffer = NULL;
                buffer->transaction = NULL;
            }
            if (buffer->async_transaction && buffer->target_node) {
                BUG_ON(!buffer->target_node->has_async_transaction);
                if (list_empty(&buffer->target_node->async_todo))
                    buffer->target_node->has_async_transaction = 0;
                else
                    list_move_tail(buffer->target_node->async_todo.next, &thread->todo);
            }
            binder_transaction_buffer_release(proc, buffer, NULL);
            binder_free_buf(proc, buffer);
            break;
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
            if (binder_debug_mask & BINDER_DEBUG_THREADS)
                printk(KERN_INFO "binder: %d:%d BC_REGISTER_LOOPER\n",
                       proc->pid, thread->pid);
            if (thread->looper & BINDER_LOOPER_STATE_ENTERED) {
                thread->looper |= BINDER_LOOPER_STATE_INVALID;
                binder_user_error("binder: %d:%d ERROR:"
                    " BC_REGISTER_LOOPER called "
                    "after BC_ENTER_LOOPER\n",
                    proc->pid, thread->pid);
            } else if (proc->requested_threads == 0) {
                thread->looper |= BINDER_LOOPER_STATE_INVALID;
                binder_user_error("binder: %d:%d ERROR:"
                    " BC_REGISTER_LOOPER called "
                    "without request\n",
                    proc->pid, thread->pid);
            } else {
                proc->requested_threads--;
                proc->requested_threads_started++;
            }
            thread->looper |= BINDER_LOOPER_STATE_REGISTERED;
            break;
        case BC_ENTER_LOOPER:
            if (binder_debug_mask & BINDER_DEBUG_THREADS)
                printk(KERN_INFO "binder: %d:%d BC_ENTER_LOOPER\n",
                       proc->pid, thread->pid);
            if (thread->looper & BINDER_LOOPER_STATE_REGISTERED) {
                thread->looper |= BINDER_LOOPER_STATE_INVALID;
                binder_user_error("binder: %d:%d ERROR:"
                    " BC_ENTER_LOOPER called after "
                    "BC_REGISTER_LOOPER\n",
                    proc->pid, thread->pid);
            }
            thread->looper |= BINDER_LOOPER_STATE_ENTERED;
            break;
        case BC_EXIT_LOOPER:
            if (binder_debug_mask & BINDER_DEBUG_THREADS)
                printk(KERN_INFO "binder: %d:%d BC_EXIT_LOOPER\n",
                       proc->pid, thread->pid);
            thread->looper |= BINDER_LOOPER_STATE_EXITED;
            break;
 
        case BC_REQUEST_DEATH_NOTIFICATION:
        case BC_CLEAR_DEATH_NOTIFICATION: {
            uint32_t target;
            void __user *cookie;
            struct binder_ref *ref;
            struct binder_ref_death *death;
 
            if (get_user(target, (uint32_t __user *)ptr))
                return -EFAULT;
            ptr += sizeof(uint32_t);
            if (get_user(cookie, (void __user * __user *)ptr))
                return -EFAULT;
            ptr += sizeof(void *);
            ref = binder_get_ref(proc, target);
            if (ref == NULL) {
                binder_user_error("binder: %d:%d %s "
                    "invalid ref %d\n",
                    proc->pid, thread->pid,
                    cmd == BC_REQUEST_DEATH_NOTIFICATION ?
                    "BC_REQUEST_DEATH_NOTIFICATION" :
                    "BC_CLEAR_DEATH_NOTIFICATION",
                    target);
                break;
            }
 
            if (binder_debug_mask & BINDER_DEBUG_DEATH_NOTIFICATION)
                printk(KERN_INFO "binder: %d:%d %s %p ref %d desc %d s %d w %d for node %d\n",
                       proc->pid, thread->pid,
                       cmd == BC_REQUEST_DEATH_NOTIFICATION ?
                       "BC_REQUEST_DEATH_NOTIFICATION" :
                       "BC_CLEAR_DEATH_NOTIFICATION",
                       cookie, ref->debug_id, ref->desc,
                       ref->strong, ref->weak, ref->node->debug_id);
 
            if (cmd == BC_REQUEST_DEATH_NOTIFICATION) {
                if (ref->death) {
                    binder_user_error("binder: %d:%"
                        "d BC_REQUEST_DEATH_NOTI"
                        "FICATION death notific"
                        "ation already set\n",
                        proc->pid, thread->pid);
                    break;
                }
                death = kzalloc(sizeof(*death), GFP_KERNEL);
                if (death == NULL) {
                    thread->return_error = BR_ERROR;
                    if (binder_debug_mask & BINDER_DEBUG_FAILED_TRANSACTION)
                        printk(KERN_INFO "binder: %d:%d "
                            "BC_REQUEST_DEATH_NOTIFICATION failed\n",
                            proc->pid, thread->pid);
                    break;
                }
                binder_stats.obj_created[BINDER_STAT_DEATH]++;
                INIT_LIST_HEAD(&death->work.entry);
                death->cookie = cookie;
                ref->death = death;
                if (ref->node->proc == NULL) {
                    ref->death->work.type = BINDER_WORK_DEAD_BINDER;
                    if (thread->looper & (BINDER_LOOPER_STATE_REGISTERED | BINDER_LOOPER_STATE_ENTERED)) {
                        list_add_tail(&ref->death->work.entry, &thread->todo);
                    } else {
                        list_add_tail(&ref->death->work.entry, &proc->todo);
                        wake_up_interruptible(&proc->wait);
                    }
                }
            } else {
                if (ref->death == NULL) {
                    binder_user_error("binder: %d:%"
                        "d BC_CLEAR_DEATH_NOTIFI"
                        "CATION death notificat"
                        "ion not active\n",
                        proc->pid, thread->pid);
                    break;
                }
                death = ref->death;
                if (death->cookie != cookie) {
                    binder_user_error("binder: %d:%"
                        "d BC_CLEAR_DEATH_NOTIFI"
                        "CATION death notificat"
                        "ion cookie mismatch "
                        "%p != %p\n",
                        proc->pid, thread->pid,
                        death->cookie, cookie);
                    break;
                }
                ref->death = NULL;
                if (list_empty(&death->work.entry)) {
                    death->work.type = BINDER_WORK_CLEAR_DEATH_NOTIFICATION;
                    if (thread->looper & (BINDER_LOOPER_STATE_REGISTERED | BINDER_LOOPER_STATE_ENTERED)) {
                        list_add_tail(&death->work.entry, &thread->todo);
                    } else {
                        list_add_tail(&death->work.entry, &proc->todo);
                        wake_up_interruptible(&proc->wait);
                    }
                } else {
                    BUG_ON(death->work.type != BINDER_WORK_DEAD_BINDER);
                    death->work.type = BINDER_WORK_DEAD_BINDER_AND_CLEAR;
                }
            }
        } break;
        case BC_DEAD_BINDER_DONE: {
            struct binder_work *w;
            void __user *cookie;
            struct binder_ref_death *death = NULL;
            if (get_user(cookie, (void __user * __user *)ptr))
                return -EFAULT;
 
            ptr += sizeof(void *);
            list_for_each_entry(w, &proc->delivered_death, entry) {
                struct binder_ref_death *tmp_death = container_of(w, struct binder_ref_death, work);
                if (tmp_death->cookie == cookie) {
                    death = tmp_death;
                    break;
                }
            }
            if (binder_debug_mask & BINDER_DEBUG_DEAD_BINDER)
                printk(KERN_INFO "binder: %d:%d BC_DEAD_BINDER_DONE %p found %p\n",
                       proc->pid, thread->pid, cookie, death);
            if (death == NULL) {
                binder_user_error("binder: %d:%d BC_DEAD"
                    "_BINDER_DONE %p not found\n",
                    proc->pid, thread->pid, cookie);
                break;
            }
 
            list_del_init(&death->work.entry);
            if (death->work.type == BINDER_WORK_DEAD_BINDER_AND_CLEAR) {
                death->work.type = BINDER_WORK_CLEAR_DEATH_NOTIFICATION;
                if (thread->looper & (BINDER_LOOPER_STATE_REGISTERED | BINDER_LOOPER_STATE_ENTERED)) {
                    list_add_tail(&death->work.entry, &thread->todo);
                } else {
                    list_add_tail(&death->work.entry, &proc->todo);
                    wake_up_interruptible(&proc->wait);
                }
            }
        } break;
 
        default:
            printk(KERN_ERR "binder: %d:%d unknown command %d\n", proc->pid, thread->pid, cmd);
            return -EINVAL;
        }
        *consumed = ptr - buffer;
    }
    return 0;
}
```

### binder_thread_read

```c
static int
binder_thread_read(struct binder_proc *proc, struct binder_thread *thread,
    void  __user *buffer, int size, signed long *consumed, int non_block)
{
    void __user *ptr = buffer + *consumed;
    void __user *end = buffer + size;
 
    int ret = 0;
    int wait_for_proc_work;
 
    if (*consumed == 0) {
        if (put_user(BR_NOOP, (uint32_t __user *)ptr))
            return -EFAULT;
        ptr += sizeof(uint32_t);
    }
 
retry:
    wait_for_proc_work = thread->transaction_stack == NULL && list_empty(&thread->todo);
 
    if (thread->return_error != BR_OK && ptr < end) {
        if (thread->return_error2 != BR_OK) {
            if (put_user(thread->return_error2, (uint32_t __user *)ptr))
                return -EFAULT;
            ptr += sizeof(uint32_t);
            if (ptr == end)
                goto done;
            thread->return_error2 = BR_OK;
        }
        if (put_user(thread->return_error, (uint32_t __user *)ptr))
            return -EFAULT;
        ptr += sizeof(uint32_t);
        thread->return_error = BR_OK;
        goto done;
    }
 
 
    thread->looper |= BINDER_LOOPER_STATE_WAITING;
    if (wait_for_proc_work)
        proc->ready_threads++;
    mutex_unlock(&binder_lock);
    if (wait_for_proc_work) {
        if (!(thread->looper & (BINDER_LOOPER_STATE_REGISTERED |
                    BINDER_LOOPER_STATE_ENTERED))) {
            binder_user_error("binder: %d:%d ERROR: Thread waiting "
                "for process work before calling BC_REGISTER_"
                "LOOPER or BC_ENTER_LOOPER (state %x)\n",
                proc->pid, thread->pid, thread->looper);
            wait_event_interruptible(binder_user_error_wait, binder_stop_on_user_error < 2);
        }
        binder_set_nice(proc->default_priority);
        if (non_block) {
            if (!binder_has_proc_work(proc, thread))
                ret = -EAGAIN;
        } else
            ret = wait_event_interruptible_exclusive(proc->wait, binder_has_proc_work(proc, thread));
    } else {
        if (non_block) {
            if (!binder_has_thread_work(thread))
                ret = -EAGAIN;
        } else
            ret = wait_event_interruptible(thread->wait, binder_has_thread_work(thread));
    }
    mutex_lock(&binder_lock);
    if (wait_for_proc_work)
        proc->ready_threads--;
    thread->looper &= ~BINDER_LOOPER_STATE_WAITING;
 
    if (ret)
        return ret;
 
    while (1) {
        uint32_t cmd;
        struct binder_transaction_data tr;
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
 
        if (end - ptr < sizeof(tr) + 4)
            break;
 
        switch (w->type) {
        case BINDER_WORK_TRANSACTION: {
            t = container_of(w, struct binder_transaction, work);
        } break;
        case BINDER_WORK_TRANSACTION_COMPLETE: {
            cmd = BR_TRANSACTION_COMPLETE;
            if (put_user(cmd, (uint32_t __user *)ptr))
                return -EFAULT;
            ptr += sizeof(uint32_t);
 
            binder_stat_br(proc, thread, cmd);
            if (binder_debug_mask & BINDER_DEBUG_TRANSACTION_COMPLETE)
                printk(KERN_INFO "binder: %d:%d BR_TRANSACTION_COMPLETE\n",
                       proc->pid, thread->pid);
 
            list_del(&w->entry);
            kfree(w);
            binder_stats.obj_deleted[BINDER_STAT_TRANSACTION_COMPLETE]++;
        } break;
        case BINDER_WORK_NODE: {
            struct binder_node *node = container_of(w, struct binder_node, work);
            uint32_t cmd = BR_NOOP;
            const char *cmd_name;
            int strong = node->internal_strong_refs || node->local_strong_refs;
            int weak = !hlist_empty(&node->refs) || node->local_weak_refs || strong;
            if (weak && !node->has_weak_ref) {
                cmd = BR_INCREFS;
                cmd_name = "BR_INCREFS";
                node->has_weak_ref = 1;
                node->pending_weak_ref = 1;
                node->local_weak_refs++;
            } else if (strong && !node->has_strong_ref) {
                cmd = BR_ACQUIRE;
                cmd_name = "BR_ACQUIRE";
                node->has_strong_ref = 1;
                node->pending_strong_ref = 1;
                node->local_strong_refs++;
            } else if (!strong && node->has_strong_ref) {
                cmd = BR_RELEASE;
                cmd_name = "BR_RELEASE";
                node->has_strong_ref = 0;
            } else if (!weak && node->has_weak_ref) {
                cmd = BR_DECREFS;
                cmd_name = "BR_DECREFS";
                node->has_weak_ref = 0;
            }
            if (cmd != BR_NOOP) {
                if (put_user(cmd, (uint32_t __user *)ptr))
                    return -EFAULT;
                ptr += sizeof(uint32_t);
                if (put_user(node->ptr, (void * __user *)ptr))
                    return -EFAULT;
                ptr += sizeof(void *);
                if (put_user(node->cookie, (void * __user *)ptr))
                    return -EFAULT;
                ptr += sizeof(void *);
 
                binder_stat_br(proc, thread, cmd);
                if (binder_debug_mask & BINDER_DEBUG_USER_REFS)
                    printk(KERN_INFO "binder: %d:%d %s %d u%p c%p\n",
                           proc->pid, thread->pid, cmd_name, node->debug_id, node->ptr, node->cookie);
            } else {
                list_del_init(&w->entry);
                if (!weak && !strong) {
                    if (binder_debug_mask & BINDER_DEBUG_INTERNAL_REFS)
                        printk(KERN_INFO "binder: %d:%d node %d u%p c%p deleted\n",
                               proc->pid, thread->pid, node->debug_id, node->ptr, node->cookie);
                    rb_erase(&node->rb_node, &proc->nodes);
                    kfree(node);
                    binder_stats.obj_deleted[BINDER_STAT_NODE]++;
                } else {
                    if (binder_debug_mask & BINDER_DEBUG_INTERNAL_REFS)
                        printk(KERN_INFO "binder: %d:%d node %d u%p c%p state unchanged\n",
                               proc->pid, thread->pid, node->debug_id, node->ptr, node->cookie);
                }
            }
        } break;
        case BINDER_WORK_DEAD_BINDER:
        case BINDER_WORK_DEAD_BINDER_AND_CLEAR:
        case BINDER_WORK_CLEAR_DEATH_NOTIFICATION: {
            struct binder_ref_death *death = container_of(w, struct binder_ref_death, work);
            uint32_t cmd;
            if (w->type == BINDER_WORK_CLEAR_DEATH_NOTIFICATION)
                cmd = BR_CLEAR_DEATH_NOTIFICATION_DONE;
            else
                cmd = BR_DEAD_BINDER;
            if (put_user(cmd, (uint32_t __user *)ptr))
                return -EFAULT;
            ptr += sizeof(uint32_t);
            if (put_user(death->cookie, (void * __user *)ptr))
                return -EFAULT;
            ptr += sizeof(void *);
            if (binder_debug_mask & BINDER_DEBUG_DEATH_NOTIFICATION)
                printk(KERN_INFO "binder: %d:%d %s %p\n",
                       proc->pid, thread->pid,
                       cmd == BR_DEAD_BINDER ?
                       "BR_DEAD_BINDER" :
                       "BR_CLEAR_DEATH_NOTIFICATION_DONE",
                       death->cookie);
 
            if (w->type == BINDER_WORK_CLEAR_DEATH_NOTIFICATION) {
                list_del(&w->entry);
                kfree(death);
                binder_stats.obj_deleted[BINDER_STAT_DEATH]++;
            } else
                list_move(&w->entry, &proc->delivered_death);
            if (cmd == BR_DEAD_BINDER)
                goto done; /* DEAD_BINDER notifications can cause transactions */
        } break;
        }
 
        if (!t)
            continue;
 
        BUG_ON(t->buffer == NULL);
        if (t->buffer->target_node) {
            struct binder_node *target_node = t->buffer->target_node;
            tr.target.ptr = target_node->ptr;
            tr.cookie =  target_node->cookie;
            t->saved_priority = task_nice(current);
            if (t->priority < target_node->min_priority &&
                !(t->flags & TF_ONE_WAY))
                binder_set_nice(t->priority);
            else if (!(t->flags & TF_ONE_WAY) ||
                 t->saved_priority > target_node->min_priority)
                binder_set_nice(target_node->min_priority);
            cmd = BR_TRANSACTION;
        } else {
            tr.target.ptr = NULL;
            tr.cookie = NULL;
            cmd = BR_REPLY;
        }
        tr.code = t->code;
        tr.flags = t->flags;
        tr.sender_euid = t->sender_euid;
 
        if (t->from) {
            struct task_struct *sender = t->from->proc->tsk;
            tr.sender_pid = task_tgid_nr_ns(sender, current->nsproxy->pid_ns);
        } else {
            tr.sender_pid = 0;
        }
 
        tr.data_size = t->buffer->data_size;
        tr.offsets_size = t->buffer->offsets_size;
        tr.data.ptr.buffer = (void *)t->buffer->data + proc->user_buffer_offset;
        tr.data.ptr.offsets = tr.data.ptr.buffer + ALIGN(t->buffer->data_size, sizeof(void *));
 
        if (put_user(cmd, (uint32_t __user *)ptr))
            return -EFAULT;
        ptr += sizeof(uint32_t);
        if (copy_to_user(ptr, &tr, sizeof(tr)))
            return -EFAULT;
        ptr += sizeof(tr);
 
        binder_stat_br(proc, thread, cmd);
        if (binder_debug_mask & BINDER_DEBUG_TRANSACTION)
            printk(KERN_INFO "binder: %d:%d %s %d %d:%d, cmd %d"
                "size %zd-%zd ptr %p-%p\n",
                   proc->pid, thread->pid,
                   (cmd == BR_TRANSACTION) ? "BR_TRANSACTION" : "BR_REPLY",
                   t->debug_id, t->from ? t->from->proc->pid : 0,
                   t->from ? t->from->pid : 0, cmd,
                   t->buffer->data_size, t->buffer->offsets_size,
                   tr.data.ptr.buffer, tr.data.ptr.offsets);
 
        list_del(&t->work.entry);
        t->buffer->allow_user_free = 1;
        if (cmd == BR_TRANSACTION && !(t->flags & TF_ONE_WAY)) {
            t->to_parent = thread->transaction_stack;
            t->to_thread = thread;
            thread->transaction_stack = t;
        } else {
            t->buffer->transaction = NULL;
            kfree(t);
            binder_stats.obj_deleted[BINDER_STAT_TRANSACTION]++;
        }
        break;
    }
 
done:
 
    *consumed = ptr - buffer;
    if (proc->requested_threads + proc->ready_threads == 0 &&
        proc->requested_threads_started < proc->max_threads &&
        (thread->looper & (BINDER_LOOPER_STATE_REGISTERED |
         BINDER_LOOPER_STATE_ENTERED)) /* the user-space code fails to */
         /*spawn a new thread if we leave this out */) {
        proc->requested_threads++;
        if (binder_debug_mask & BINDER_DEBUG_THREADS)
            printk(KERN_INFO "binder: %d:%d BR_SPAWN_LOOPER\n",
                   proc->pid, thread->pid);
        if (put_user(BR_SPAWN_LOOPER, (uint32_t __user *)buffer))
            return -EFAULT;
    }
    return 0;
}
```



## 来源

[Binder内核驱动源码分析 (zenki2001cn.github.io)](http://zenki2001cn.github.io/Wiki/Android/Binder内核驱动源码分析.html)



