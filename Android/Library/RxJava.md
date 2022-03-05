<h1 align="center">RxJava</h1>

[toc]

## RxJava到底是什么？

RxJava是基于Java虚拟机上的响应式扩展库，它通过**使用可观察的序列将异步和基于事件的程序组合起来**。 与此同时，它**扩展了观察者模式来支持数据/事件序列**，并且添加了操作符，这些**操作符允许你声明性地组合序列**，同时抽象出要关注的问题：比如低级线程、同步、线程安全和并发数据结构等。

从RxJava的官方定义来看，我们如果要想真正地理解RxJava，就必须对它以下两个部分进行深入的分析：

1. **订阅流程**

2. **线程切换**

当然，RxJava操作符的源码也是很不错的学习资源，特别是FlatMap、Zip等操作符的源码，有很多可以借鉴的地方，但是它们内部的实现比较复杂。

## RxJava的订阅流程

首先给出RxJava消息订阅的例子：

```java
Observable.create(newObservableOnSubscribe<String>() {
    @Override
    public void subscribe(ObservableEmitter<String>emitter) throws Exception {
        emitter.onNext("1");
        emitter.onNext("2");
        emitter.onNext("3");
        emitter.onComplete();
    }
}).subscribe(new Observer<String>() {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "onSubscribe");
    }
    @Override
    public void onNext(String s) {
        Log.d(TAG, "onNext : " + s);
    }
    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "onError : " + e.toString());
    }
    @Override
    public void onComplete() {
        Log.d(TAG, "onComplete");
    }
});
```

可以看到，这里首先创建了一个被观察者，然后创建一个观察者订阅了这个被观察者，因此下面分两个部分对RxJava的订阅流程进行分析：

1. **创建被观察者过程**

2. **订阅过程**

### 创建被观察者过程

首先，上面使用了Observable类的create()方法创建了一个被观察者，看看里面做了什么。

#### Observable#create()

```java
// 省略一些检测性的注解
public static <T> Observable<T> create(ObservableOnSubscribe<T> source) {
    ObjectHelper.requireNonNull(source, "source is null");
    return RxJavaPlugins.onAssembly(new ObservableCreate<T>(source));
}
```

在Observable的create()里面实际上是创建了一个新的ObservableCreate对象，同时，把我们定义好的ObservableOnSubscribe对象传入了ObservableCreate对象中，最后调用了RxJavaPlugins.onAssembly()方法。接下来看看这个ObservableCreate是干什么的。

#### ObservableCreate

```java
public final class ObservableCreate<T> extends Observable<T> {

    final ObservableOnSubscribe<T> source;

    public ObservableCreate(ObservableOnSubscribe<T> source) {
        this.source = source;
    }
    
    ...
}
```

这里仅仅是把ObservableOnSubscribe这个对象保存在ObservableCreate中了。然后看看RxJavaPlugins.onAssembly()这个方法的处理。

#### RxJavaPlugins#onAssembly()

```java
public static <T> Observable<T> onAssembly(@NonNull Observable<T> source) {

    // 应用hook函数的一些处理，一般用到不到
    ...
    return source;
}
```

最终仅仅是把我们的ObservableCreate给返回了。

#### 创建被观察者过程小结

从以上分析可知，Observable.create()方法仅仅是**先将我们自定义的ObservableOnSubscribe对象重新包装成了一个ObservableCreate对象**。

### 订阅过程

接着，看看Observable.subscribe()的订阅过程是如何实现的。

#### Observable#subscribe()

```java
public final void subscribe(Observer<? super T> observer) {
    ...
    
    // 1
    observer = RxJavaPlugins.onSubscribe(this,observer);
    
    ...
    
    // 2
    subscribeActual(observer);
    
    ...
}
```

在注释1处，在Observable的subscribe()方法内部首先调用了RxJavaPlugins的onSubscribe()方法。

#### RxJavaPlugins#onSubscribe()

```java
public static <T> Observer<? super T> onSubscribe(@NonNull Observable<T> source, @NonNull Observer<? super T> observer) {

    // 应用hook函数的一些处理，一般用到不到
    ...
    
    return observer;
}
```

除去hook应用的逻辑，这里仅仅是将observer返回了。接着来分析下注释2处的subscribeActual()方法，

#### Observable#subscribeActual()

```java
protected abstract void subscribeActual(Observer<? super T> observer);
```

这是一个抽象的方法，很明显，它对应的具体实现类就是我们在第一步创建的ObservableCreate类，接下来看到ObservableCreate的subscribeActual()方法。

#### ObservableCreate#subscribeActual()

```java
@Override
protected void subscribeActual(Observer<? super T> observer) {
    // 1
    CreateEmitter<T> parent = new CreateEmitter<T>(observer);
    // 2
    observer.onSubscribe(parent);

    try {
        // 3
        source.subscribe(parent);
    } catch (Throwable ex) {
        Exceptions.throwIfFatal(ex);
        parent.onError(ex);
    }
}
```

在注释1处，首先新创建了一个CreateEmitter对象，同时传入了我们自定义的observer对象进去。

##### CreateEmitter

```java
static final class CreateEmitter<T>
extends AtomicReference<Disposable>
implements ObservableEmitter<T>, Disposable {

    ...
    
    final Observer<? super T> observer;

    CreateEmitter(Observer<? super T> observer) {
        this.observer = observer;
    }
    
    ...
}
```

从上面可以看出，**CreateEmitter通过继承了Java并发包中的原子引用类AtomicReference保证了事件流切断状态Dispose的一致性**（这里不理解的话，看到后面讲解Dispose的时候就明白了），并**实现了ObservableEmitter接口和Disposable接口**，接着我们分析下注释2处的observer.onSubscribe(parent)，这个onSubscribe回调的含义其实就是**告诉观察者已经成功订阅了被观察者**。再看到注释3处的source.subscribe(parent)这行代码，这里的source其实是ObservableOnSubscribe对象，我们看到ObservableOnSubscribe的subscribe()方法。

##### ObservableOnSubscribe#subscribe()

```java
Observable observable = Observable.create(new ObservableOnSubscribe<String>() {
    @Override
    public voidsubscribe(ObservableEmitter<String> emitter) throws Exception {
        emitter.onNext("1");
        emitter.onNext("2");
        emitter.onNext("3");
        emitter.onComplete();
    }
});
```

这里面使用到了ObservableEmitter的onNext()方法将事件流发送出去，最后调用了onComplete()方法完成了订阅过程。ObservableEmitter是一个抽象类，实现类就是我们传入的CreateEmitter对象，接下来我们看看CreateEmitter的onNext()方法和onComplete()方法的处理。

##### CreateEmitter#onNext() && CreateEmitter#onComplete()

```java
static final class CreateEmitter<T>
extends AtomicReference<Disposable>
implements ObservableEmitter<T>, Disposable {

...

@Override
public void onNext(T t) {
    ...
    
    if (!isDisposed()) {
        //调用观察者的onNext()
        observer.onNext(t);
    }
}

@Override
public void onComplete() {
    if (!isDisposed()) {
        try {
            observer.onComplete();
        } finally {
            dispose();
        }
    }
}


...

}
```

在CreateEmitter的onNext和onComplete方法中首先都要经过一个**isDisposed**的判断，作用就是看**当前的事件流是否被切断（废弃）掉了**，默认是不切断的，如果想要切断，可以调用Disposable的dispose()方法将此状态设置为切断（废弃）状态。继续看看这个isDisposed内部的处理。

##### ObservableEmitter#isDisposed()

```java
@Override
public boolean isDisposed() {
    return DisposableHelper.isDisposed(get());
}
```

注意到这里通过get()方法首先从ObservableEmitter的AtomicReference中拿到了保存的Disposable状态。然后交给了DisposableHelper进行判断处理。接下来看看DisposableHelper的处理。

##### DisposableHelper#isDisposed() && DisposableHelper#set()

```java
public enum DisposableHelper implements Disposable {

    DISPOSED;

    public static boolean isDisposed(Disposable d) {
        // 1
        return d == DISPOSED;
    }
    
    public static boolean set(AtomicReference<Disposable> field, Disposable d) {
        for (;;) {
            Disposable current = field.get();
            if (current == DISPOSED) {
                if (d != null) {
                    d.dispose();
                }
                return false;
            }
            // 2
            if (field.compareAndSet(current, d)) {
                if (current != null) {
                    current.dispose();
                }
                return true;
            }
        }
    }
    
    ...
    
    public static boolean dispose(AtomicReference<Disposable> field) {
        Disposable current = field.get();
        Disposable d = DISPOSED;
        if (current != d) {
            // ...
            current = field.getAndSet(d);
            if (current != d) {
                if (current != null) {
                    current.dispose();
                }
                return true;
            }
        }
        return false;
    }
    
    ...
}
```

DisposableHelper是一个枚举类，内部只有一个值即DISPOSED, 从上面的分析可知它就是用来**标记事件流被切断（废弃）状态的**。先看到注释2和注释3处的代码**field.compareAndSet(current, d)和field.getAndSet(d)**，这里使用了**原子引用AtomicReference内部包装的[CAS](https://www.jianshu.com/p/ab2c8fce878b)方法处理了标志Disposable的并发读写问题**。最后看到注释3处，将我们传入的CreateEmitter这个原子引用类保存的Dispable状态和DisposableHelper内部的DISPOSED进行比较，如果相等，就证明数据流被切断了。为了更进一步理解Disposed的作用，再来看看CreateEmitter中剩余的关键方法。

##### CreateEmitter

```java
@Override
public void onNext(T t) {
    ...
    // 1
    if (!isDisposed()) {
        observer.onNext(t);
    }
}

@Override
public void onError(Throwable t) {
    if (!tryOnError(t)) {
        // 2
        RxJavaPlugins.onError(t);
    }
}

@Override
public boolean tryOnError(Throwable t) {
    ...
    // 3
    if (!isDisposed()) {
        try {
            observer.onError(t);
        } finally {
            // 4
            dispose();
        }
        return true;
    }
    return false;
}

@Override
public void onComplete() {
    // 5
    if (!isDisposed()) {
        try {
            observer.onComplete();
        } finally {
            // 6
            dispose();
        }
    }
}
```

在注释1、3、5处，onNext()和onError()、onComplete()方法首先都会判断事件流是否被切断，如果事件流此时被切断了，那么onNext()和onComplete()则会退出方法体，不做处理，**onError()则会执行到RxJavaPlugins.onError(t)这句代码，内部会直接抛出异常，导致崩溃**。如果事件流没有被切断，那么在onError()和onComplete()内部最终会调用到注释4、6处的这句dispose()代码，将事件流进行切断，由此可知，**onError()和onComplete()只能调用一个，如果先执行的是onComplete()，再调用onError()的话就会导致异常崩溃**。

## RxJava的线程切换

首先给出RxJava线程切换的例子：

```java
Observable.create(new ObservableOnSubscribe<String>() {
    @Override
    public voidsubscribe(ObservableEmitter<String>emitter) throws Exception {
        emitter.onNext("1");
        emitter.onNext("2");
        emitter.onNext("3");
        emitter.onComplete();
    }
}) 
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(new Observer<String>() {
        @Override
        public void onSubscribe(Disposable d) {
            Log.d(TAG, "onSubscribe");
        }
        @Override
        public void onNext(String s) {
            Log.d(TAG, "onNext : " + s);
        }
        @Override
        public void onError(Throwable e) {
            Log.d(TAG, "onError : " +e.toString());
        }
        @Override
        public void onComplete() {
            Log.d(TAG, "onComplete");
        }
});
```

可以看到，RxJava的线程切换主要**分为subscribeOn()和observeOn()方法**，首先，来分析下subscribeOn()方法。

### subscribeOn(Schedulers.io())

在Schedulers.io()方法中，我们需要先传入一个Scheduler调度类，这里是传入了一个调度到io子线程的调度类，我们看看这个Schedulers.io()方法内部是怎么构造这个调度器的。

### Schedulers#io()

```java
static final Scheduler IO;

...

public static Scheduler io() {
    // 1
    return RxJavaPlugins.onIoScheduler(IO);
}

static {
    ...

    // 2
    IO = RxJavaPlugins.initIoScheduler(new IOTask());
}

static final class IOTask implements Callable<Scheduler> {
    @Override
    public Scheduler call() throws Exception {
        // 3
        return IoHolder.DEFAULT;
    }
}

static final class IoHolder {
    // 4
    static final Scheduler DEFAULT = new IoScheduler();
}
```

Schedulers这个类的代码很多，这里我只拿出有关Schedulers.io这个方法涉及的逻辑代码进行讲解。首先，在注释1处，同前面分析的订阅流程的处理一样，只是一个处理hook的逻辑，最终返回的还是传入的这个IO对象。再看到注释2处，**在Schedulers的静态代码块中将IO对象进行了初始化，其实质就是新建了一个IOTask的静态内部类**，在IOTask的call方法中，也就是注释3处，可以了解到使用了静态内部类的方式把创建的IOScheduler对象给返回出去了。绕了这么大圈子，**Schedulers.io方法其实质就是返回了一个IOScheduler对象**。

### Observable#subscribeOn()

```java
  public final Observable<T> subscribeOn(Scheduler scheduler) {
    ...
    
    return RxJavaPlugins.onAssembly(new ObservableSubscribeOn<T>(this, scheduler));
}
```

在subscribeOn()方法里面，又将ObservableCreate包装成了一个ObservableSubscribeOn对象。我们关注到ObservableSubscribeOn类。

### ObservableSubscribeOn

```java
public final class ObservableSubscribeOn<T> extends AbstractObservableWithUpstream<T, T> {
    final Scheduler scheduler;

    public ObservableSubscribeOn(ObservableSource<T> source, Scheduler scheduler) {
        // 1
        super(source);
        this.scheduler = scheduler;
    }

    @Override
    public void subscribeActual(final Observer<? super T> observer) {
        // 2
        final SubscribeOnObserver<T> parent = new SubscribeOnObserver<T>(observer);
        
        // 3
        observer.onSubscribe(parent);
        
        // 4
        parent.setDisposable(scheduler.scheduleDirect(new SubscribeTask(parent)));
    }

...
}
```

首先，在注释1处，将传进来的source和scheduler保存起来。接着，等到实际订阅的时候，就会执行到这个subscribeActual方法，在注释2处，将我们自定义的Observer包装成了一个SubscribeOnObserver对象。在注释3处，通知观察者订阅了被观察者。在注释4处，内部先创建了一个SubscribeTask对象，来看看它的实现。

### ObservableSubscribeOn#SubscribeTask

```java
final class SubscribeTask implements Runnable {
    private final SubscribeOnObserver<T> parent;

    SubscribeTask(SubscribeOnObserver<T> parent) {
        this.parent = parent;
    }

    @Override
    public void run() {
        source.subscribe(parent);
    }
}
```

SubscribeTask是ObservableSubscribeOn的内部类，它实质上就是一个任务类，在它的run方法中会执行到source.subscribe(parent)的订阅方法，**这个source其实就是我们在ObservableSubscribeOn构造方法中传进来的ObservableCreate对象**。接下来看看scheduler.scheduleDirect()内部的处理。

### Scheduler#scheduleDirect()

```java
public Disposable scheduleDirect(@NonNull Runnable run) {
    return scheduleDirect(run, 0L, TimeUnit.NANOSECONDS);
}

public Disposable scheduleDirect(@NonNull Runnable run, long delay, @NonNull TimeUnit unit) {

    // 1
    final Worker w = createWorker();

    // 2
    final Runnable decoratedRun = RxJavaPlugins.onSchedule(run);

    // 3
    DisposeTask task = new DisposeTask(decoratedRun, w);

    // 4
    w.schedule(task, delay, unit);

    return task;
}
```

这里最后会执行到上面这个scheduleDirect()重载方法。首先，在注释1处，会调用createWorker()方法创建一个工作者对象Worker，它是一个抽象类，这里的实现类就是IoScheduler，下面看看IoScheduler类的createWorker()方法。

#### IOScheduler#createWorker()

```java
final AtomicReference<CachedWorkerPool> pool;

...

public IoScheduler(ThreadFactory threadFactory) {
    this.threadFactory = threadFactory;
    this.pool = new AtomicReference<CachedWorkerPool>(NONE);
    start();
}

...

@Override
public Worker createWorker() {
    // 1
    return new EventLoopWorker(pool.get());
}

static final class EventLoopWorker extends Scheduler.Worker {
    ...

    EventLoopWorker(CachedWorkerPool pool) {
        this.pool = pool;
        this.tasks = new CompositeDisposable();
        // 2
        this.threadWorker = pool.get();
    }
    
}
```

首先，在注释1处调用了pool.get()这个方法，**pool是一个CachedWorkerPool类型的原子引用对象**，它的作用就是**用于缓存工作者对象Worker的**。然后，将得到的CachedWorkerPool传入新创建的EventLoopWorker对象中。重点关注一下注释2处，这里将CachedWorkerPool缓存的threadWorker对象保存起来了。

下面继续分析3.6处代码段的注释2处的代码，这里又是一个关于hook的封装处理，最终还是返回的当前的Runnable对象。在注释3处新建了一个切断任务DisposeTask将decoratedRun和w对象包装了起来。最后在注释4处调用了工作者的schedule()方法。下面来分析下它内部的处理。

#### IoScheduler#schedule()

```java
@Override
public Disposable schedule(@NonNull Runnableaction, long delayTime, @NonNull TimeUnit unit){
    ...
    
    return threadWorker.scheduleActual(action,delayTime, unit, tasks);
}
```

内部调用了threadWorker的scheduleActual()方法，实际上是调用到了父类NewThreadWorker的scheduleActual()方法，继续看看NewThreadWorker的scheduleActual()方法中做的事情。

#### NewThreadWorker#scheduleActual()

```java
public NewThreadWorker(ThreadFactory threadFactory) {
    executor = SchedulerPoolFactory.create(threadFactory);
}


@NonNull
public ScheduledRunnable scheduleActual(final Runnable run, long delayTime, @NonNull TimeUnit unit, @Nullable DisposableContainer parent) {
    Runnable decoratedRun = RxJavaPlugins.onSchedule(run);

    // 1
    ScheduledRunnable sr = new ScheduledRunnable(decoratedRun, parent);
    
   
    if (parent != null) {
        if (!parent.add(sr)) {
            return sr;
        }
    }

    Future<?> f;
    try {
        // 2
        if (delayTime <= 0) {
            // 3
            f = executor.submit((Callable<Object>)sr);
        } else {
            // 4
            f = executor.schedule((Callable<Object>)sr, delayTime, unit);
        }
        sr.setFuture(f);
    } catch (RejectedExecutionException ex) {
        if (parent != null) {
            parent.remove(sr);
        }
        RxJavaPlugins.onError(ex);
    }

    return sr;
}
```

在NewThreadWorker的scheduleActual()方法的内部，在注释1处首先会新建一个ScheduledRunnable对象，将Runnable对象和parent包装起来了，**这里parent是一个DisposableContainer对象，它实际的实现类是CompositeDisposable类，它是一个保存所有事件流是否被切断状态的容器，其内部的实现是使用了RxJava自己定义的一个简单的OpenHashSet类进行存储**。最后注释2处，判断是否设置了延迟时间，如果设置了，则调用线程池的submit()方法立即进行线程切换，否则，调用schedule()方法进行延时执行线程切换。

### 为什么多次执行subscribeOn()，只有第一次有效？

从上面的分析，可以很容易了解到**被观察者被订阅时是从最外面的一层（ObservableSubscribeOn）通知到里面的一层（ObservableOnSubscribe）**，当连续执行了到多次subscribeOn()的时候，其实就是先执行倒数第一次的subscribeOn()方法，直到最后一次执行的subscribeOn()方法，这样肯定会覆盖前面的线程切换。

### observeOn(AndroidSchedulers.mainThread())

```java
public final Observable<T> observeOn(Scheduler scheduler) {
    return observeOn(scheduler, false, bufferSize());
}

public final Observable<T> observeOn(Scheduler scheduler, boolean delayError, int bufferSize) {
    ....
    
    return RxJavaPlugins.onAssembly(new ObservableObserveOn<T>(this, scheduler, delayError, bufferSize));
}
```

可以看到，observeOn()方法内部最终也是返回了一个ObservableObserveOn对象，直接来看看ObservableObserveOn的subscribeActual()方法。

### ObservableObserveOn#subscribeActual()

```java
@Override
protected void subscribeActual(Observer<? super T> observer) {
    // 1
    if (scheduler instanceof TrampolineScheduler) {
        // 2
        source.subscribe(observer);
    } else {
        // 3
        Scheduler.Worker w = scheduler.createWorker();
        // 4
        source.subscribe(new ObserveOnObserver<T>(observer, w, delayError, bufferSize));
    }
}
```

首先，在注释1处，判断指定的调度器是不是TrampolineScheduler，这是一个不进行线程切换，立即执行当前代码的调度器。如果是，则会直接调用ObservableSubscribeOn的subscribe()方法，如果不是，则会在注释3处创建一个工作者对象。然后，在注释4处创建一个新的ObserveOnObserver将SubscribeOnobserver对象包装起来，并传入ObservableSubscribeOn的subscribe()方法进行订阅。接下来看看ObserveOnObserver类的重点方法。

### ObserveOnObserver

```java
@Override
public void onNext(T t) {
    ...
    if (sourceMode != QueueDisposable.ASYNC) {
        // 1
        queue.offer(t);
    }
    schedule();
}

@Override
public void onError(Throwable t) {
    ...
    schedule();
}

@Override
public void onComplete() {
    ...
    schedule();
}

```

去除非主线逻辑的代码，在ObserveOnObserver的onNext()和onError()、onComplete()方法中最后都会调用到schedule()方法。接着看schedule()方法，其中**onNext()还会把消息存放到队列中**。

### ObserveOnObserver#schedule()

```java
void schedule() {
    if (getAndIncrement() == 0) {
        worker.schedule(this);
    }
}
```

这里使用了worker进行调度ObserveOnObserver这个实现了Runnable的任务。worker就是在AndroidSchedulers.mainThread()中创建的，内部其实就是**使用Handler进行线程切换的**，此处不再赘述了。接着看ObserveOnObserver的run()方法。

### ObserveOnObserver#run()

```java
@Override
public void run() {
    // 1
    if (outputFused) {
        drainFused();
    } else {
        // 2
        drainNormal();
    }
}
```

在注释1处会**先判断outputFused这个标志位，它表示事件流是否被融化掉，默认是false，所以，最后会执行到drainNormal()方法**。接着看看drainNormal()方法内部的处理。

### ObserveOnObserver#drainNormal()

```java
void drainNormal() {
    int missed = 1;
    
    final SimpleQueue<T> q = queue;
    
    // 1
    final Observer<? super T> a = downstream;
    
    ...
    
    // 2
    v = q.poll();
    
    ...
    // 3
    a.onNext(v);
    
    ...
}
```

在注释1处，这里的downstream实际上是从外面传进来的SubscribeOnObserver对象。在注释2处将队列中的消息取出来，接着在注释3处调用了SubscribeOnObserver的onNext方法。**最终，会从我们包装类的最外层一直调用到最里面的我们自定义的Observer中的onNext()方法，所以，在observeOn()方法下面的链式代码都会执行到它所指定的线程中，噢，原来如此**。



> 很多人使用RxJava也已经挺长时间了，但是一直没有去深入去了解过它的内部实现原理，**如今细细品尝，的确是酣畅淋漓**。