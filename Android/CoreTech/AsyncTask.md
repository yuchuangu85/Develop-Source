<h1 align="center">AsyncTask</h1>

[toc]

## AsyncTask

AsyncTask 封装了 Thread 和 Handler，并不适合特别耗时的后台任务，对于特别耗时的任务来说，建议使用线程池。

### 基本使用

| 方法                            | 说明                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| onPreExecute()                  | 异步任务执行前调用，用于做一些准备工作                       |
| doInBackground(Params...params) | 用于执行异步任务，此方法中可以通过 publishProgress 方法来更新任务的进度，publishProgress 会调用 onProgressUpdate 方法 |
| onProgressUpdate                | 在主线程中执行，后台任务的执行进度发生改变时调用             |
| onPostExecute                   | 在主线程中执行，在异步任务执行之后                           |

```java
import android.os.AsyncTask;

public class DownloadTask extends AsyncTask<String, Integer, Boolean> {

    @Override
    protected void onPreExecute() {
        super.onPreExecute();
    }

    @Override
    protected Boolean doInBackground(String... strings) {
        return null;
    }

    @Override
    protected void onProgressUpdate(Integer... values) {
        super.onProgressUpdate(values);
    }

    @Override
    protected void onPostExecute(Boolean aBoolean) {
        super.onPostExecute(aBoolean);
    }
}
```

- 异步任务的实例必须在 UI 线程中创建，即 AsyncTask 对象必须在UI线程中创建。
- execute(Params... params)方法必须在UI线程中调用。
- 不要手动调用 onPreExecute()，doInBackground()，onProgressUpdate()，onPostExecute() 这几个方法。
- 不能在 doInBackground() 中更改UI组件的信息。
- 一个任务实例只能执行一次，如果执行第二次将会抛出异常。
- execute() 方法会让同一个进程中的 AsyncTask 串行执行，如果需要并行，可以调用 executeOnExcutor 方法。

### 工作原理

``AsyncTask.java``

```java
@MainThread
public final AsyncTask<Params, Progress, Result> execute(Params... params) {
    return executeOnExecutor(sDefaultExecutor, params);
}

@MainThread
public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
        Params... params) {
    if (mStatus != Status.PENDING) {
        switch (mStatus) {
            case RUNNING:
                throw new IllegalStateException("Cannot execute task:"
                        + " the task is already running.");
            case FINISHED:
                throw new IllegalStateException("Cannot execute task:"
                        + " the task has already been executed "
                        + "(a task can be executed only once)");
        }
    }

    mStatus = Status.RUNNING;

    onPreExecute();

    mWorker.mParams = params;
    exec.execute(mFuture);

    return this;
}
```

sDefaultExecutor 是一个串行的线程池，一个进程中的所有的 AsyncTask 全部在该线程池中执行。AysncTask 中有两个线程池（SerialExecutor 和 THREAD_POOL_EXECUTOR）和一个 Handler（InternalHandler），其中线程池 SerialExecutor 用于任务的排队，THREAD_POOL_EXECUTOR 用于真正地执行任务，InternalHandler 用于将执行环境从线程池切换到主线程。

``AsyncTask.java``

```java
private static Handler getMainHandler() {
    synchronized (AsyncTask.class) {
        if (sHandler == null) {
            sHandler = new InternalHandler(Looper.getMainLooper());
        }
        return sHandler;
    }
}

private static class InternalHandler extends Handler {
    public InternalHandler(Looper looper) {
        super(looper);
    }

    @SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
    @Override
    public void handleMessage(Message msg) {
        AsyncTaskResult<?> result = (AsyncTaskResult<?>) msg.obj;
        switch (msg.what) {
            case MESSAGE_POST_RESULT:
                // There is only one result
                result.mTask.finish(result.mData[0]);
                break;
            case MESSAGE_POST_PROGRESS:
                result.mTask.onProgressUpdate(result.mData);
                break;
        }
    }
}


private Result postResult(Result result) {
    @SuppressWarnings("unchecked")
    Message message = getHandler().obtainMessage(MESSAGE_POST_RESULT,
            new AsyncTaskResult<Result>(this, result));
    message.sendToTarget();
    return result;
}
```



## 总结

Android UI是线程不安全的，如果想要在子线程里进行UI操作，就需要借助Android的异步消息处理机制。

不过为了更加方便我们在子线程中更新UI元素，Android从1.5版本就引入了一个AsyncTask类

在Android当中，通常将线程分为两种，一种叫做Main Thread，除了Main Thread之外的线程都可称为Worker Thread

AsyncTask：异步任务，从字面上来说，就是在我们的UI主线程运行的时候，异步的完成一些操作。AsyncTask允许我们的执行一个异步的任务在后台。我们可以将耗时的操作放在异步任务当中来执行，并随时将任务执行的结果返回给我们的UI线程来更新我们的UI控件

我们定义一个类来继承AsyncTask这个类的时候，我们需要为其指定3个泛型参数:

AsyncTask　<Params, Progress, Result>

Params: 这个泛型指定的是我们传递给异步任务执行时的参数的类型

Progress: 这个泛型指定的是我们的异步任务在执行的时候将执行的进度返回给UI线程的参数的类型

Result: 这个泛型指定的异步任务执行完后返回给UI线程的结果的类型

如果都不指定的话，则都将其写成Void

执行一个异步任务的时候，其需要按照下面的4个步骤分别执行:

onPreExecute(): 这个方法是在执行异步任务之前的时候执行，并且是在UI Thread当中执行的，通常我们在这个方法里做一些UI控件的初始化的操作，例如弹出要给ProgressDialog

doInBackground(Params... params): 在onPreExecute()方法执行完之后，会马上执行这个方法，这个方法就是来处理异步任务的方法，Android操作系统会在后台的线程池当中开启一个worker thread来执行我们的这个方法，所以这个方法是在worker thread当中执行的，这个方法执行完之后就可以将我们的执行结果发送给我们的最后一个 onPostExecute 方法，在这个方法里，我们可以从网络当中获取数据等一些耗时的操作

onProgressUpdate(Progess... values): 这个方法也是在UI Thread当中执行的，我们在异步任务执行的时候，有时候需要将执行的进度返回给我们的UI界面，例如下载一张网络图片，我们需要时刻显示其下载的进度，就可以使用这个方法来更新我们的进度。这个方法在调用之前，我们需要在 doInBackground 方法中调用一个 publishProgress(Progress) 的方法来将我们的进度时时刻刻传递给 onProgressUpdate 方法来更新

onPostExecute(Result... result): 当我们的异步任务执行完之后，就会将结果返回给这个方法，这个方法也是在UI Thread当中调用的，我们可以将返回的结果显示在UI控件上

可以在任何时刻来取消我们的异步任务的执行，通过调用 cancel(boolean)方法.

在使用AsyncTask做异步任务的时候必须要遵循的原则：

AsyncTask类必须在UI Thread当中加载，在Android Jelly_Bean版本后这些都是自动完成的

AsyncTask的对象必须在UI Thread当中实例化

execute方法必须在UI Thread当中调用

不要手动的去调用AsyncTask的onPreExecute, doInBackground, publishProgress, onProgressUpdate, onPostExecute方法，这些都是由Android系统自动调用的

AsyncTask任务只能被执行一次

## 优点、缺点:

* 优点
  * 简单,快捷
  * 过程可控

* 缺点
  * 在使用多个异步操作和并需要进行Ui变更时,就变得复杂起来.