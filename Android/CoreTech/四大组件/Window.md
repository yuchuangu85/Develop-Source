<h1 align="center">Window</h1>

[toc]

## Window 概念与分类

Window 是一个抽象类，它的具体实现是 PhoneWindow。WindowManager 是外界访问 Window 的入口，Window 的具体实现位于 WindowManagerService 中，WindowManager 和 WindowManagerService 的交互是一个 IPC 过程。Android 中所有的视图都是通过 Window 来呈现，因此 Window 实际是 View 的直接管理者。

| Window 类型        | 说明                                               | 层级      |
| ------------------ | -------------------------------------------------- | --------- |
| Application Window | 对应着一个 Activity                                | 1~99      |
| Sub Window         | 不能单独存在，只能附属在父 Window 中，如 Dialog 等 | 1000~1999 |
| System Window      | 需要权限声明，如 Toast 和 系统状态栏等             | 2000~2999 |

## Window 的内部机制

Window 是一个抽象的概念，每一个 Window 对应着一个 View 和一个 ViewRootImpl。Window 实际是不存在的，它是以 View 的形式存在。对 Window 的访问必须通过 WindowManager，WindowManager 的实现类是 WindowManagerImpl：

``WindowManagerImpl.java``

```java
@Override
public void addView(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
    applyDefaultToken(params);
    mGlobal.addView(view, params, mContext.getDisplay(), mParentWindow);
}

@Override
public void updateViewLayout(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
    applyDefaultToken(params);
    mGlobal.updateViewLayout(view, params);
}

@Override
public void removeView(View view) {
    mGlobal.removeView(view, false);
}
```

WindowManagerImpl 没有直接实现 Window 的三大操作，而是全部交给 WindowManagerGlobal 处理，WindowManagerGlobal 以工厂的形式向外提供自己的实例：

``WindowManagerGlobal.java``

```java
// 添加
public void addView(View view, ViewGroup.LayoutParams params,
        Display display, Window parentWindow) {
    ···
    // 子 Window 的话需要调整一些布局参数
    final WindowManager.LayoutParams wparams = (WindowManager.LayoutParams) params;
    if (parentWindow != null) {
        parentWindow.adjustLayoutParamsForSubWindow(wparams);
    } else {
        ···
    }
    ViewRootImpl root;
    View panelParentView = null;
    synchronized (mLock) {
        // 新建一个 ViewRootImpl，并通过其 setView 来更新界面完成 Window 的添加过程
        ···
        root = new ViewRootImpl(view.getContext(), display);
        view.setLayoutParams(wparams);
        mViews.add(view);
        mRoots.add(root);
        mParams.add(wparams);
        // do this last because it fires off messages to start doing things
        try {
            root.setView(view, wparams, panelParentView);
        } catch (RuntimeException e) {
            // BadTokenException or InvalidDisplayException, clean up.
            if (index >= 0) {
                removeViewLocked(index, true);
            }
            throw e;
        }
    }
}

// 删除
@UnsupportedAppUsage
public void removeView(View view, boolean immediate) {
    ···
    synchronized (mLock) {
        int index = findViewLocked(view, true);
        View curView = mRoots.get(index).getView();
        removeViewLocked(index, immediate);
        ···
    }
}

private void removeViewLocked(int index, boolean immediate) {
    ViewRootImpl root = mRoots.get(index);
    View view = root.getView();
    if (view != null) {
        InputMethodManager imm = InputMethodManager.getInstance();
        if (imm != null) {
            imm.windowDismissed(mViews.get(index).getWindowToken());
        }
    }
    boolean deferred = root.die(immediate);
    if (view != null) {
        view.assignParent(null);
        if (deferred) {
            mDyingViews.add(view);
        }
    }
}

// 更新
public void updateViewLayout(View view, ViewGroup.LayoutParams params) {
    ···
    final WindowManager.LayoutParams wparams = (WindowManager.LayoutParams)params;
    view.setLayoutParams(wparams);
    synchronized (mLock) {
        int index = findViewLocked(view, true);
        ViewRootImpl root = mRoots.get(index);
        mParams.remove(index);
        mParams.add(index, wparams);
        root.setLayoutParams(wparams, false);
    }
}
```

在 ViewRootImpl 中最终会通过 WindowSession 来完成 Window 的添加、更新、删除工作，mWindowSession 的类型是 IWindowSession，是一个 Binder 对象，真正地实现类是 Session，是一个 IPC 过程。

## Window 的创建过程

### Activity 的 Window 创建过程

在 Activity 的创建过程中，最终会由 ActivityThread 的 performLaunchActivity() 来完成整个启动过程，该方法内部会通过类加载器创建 Activity 的实例对象，并调用 attach 方法关联一系列上下文环境变量。在 Activity 的 attach 方法里，系统会创建所属的 Window 对象并设置回调接口，然后在 Activity 的 setContentView 方法中将视图附属在 Window 上：

``Activity.java``

```java
final void attach(Context context, ActivityThread aThread,
        Instrumentation instr, IBinder token, int ident,
        Application application, Intent intent, ActivityInfo info,
        CharSequence title, Activity parent, String id,
        NonConfigurationInstances lastNonConfigurationInstances,
        Configuration config, String referrer, IVoiceInteractor voiceInteractor,
        Window window, ActivityConfigCallback activityConfigCallback) {
    attachBaseContext(context);

    mFragments.attachHost(null /*parent*/);

    mWindow = new PhoneWindow(this, window, activityConfigCallback);
    mWindow.setWindowControllerCallback(this);
    mWindow.setCallback(this);
    mWindow.setOnWindowDismissedCallback(this);
    mWindow.getLayoutInflater().setPrivateFactory(this);
    if (info.softInputMode != WindowManager.LayoutParams.SOFT_INPUT_STATE_UNSPECIFIED) {
        mWindow.setSoftInputMode(info.softInputMode);
    }
    if (info.uiOptions != 0) {
        mWindow.setUiOptions(info.uiOptions);
    }
    ···
}
···

public void setContentView(@LayoutRes int layoutResID) {
    getWindow().setContentView(layoutResID);
    initWindowDecorActionBar();
}

```

``PhoneWindow.java``

```java
@Override
public void setContentView(int layoutResID) {
    if (mContentParent == null) { // 如果没有 DecorView，就创建
        installDecor();
    } else {
        mContentParent.removeAllViews();
    }
    mLayoutInflater.inflate(layoutResID, mContentParent);
    final Callback cb = getCallback();
    if (cb != null && !isDestroyed()) {
        // 回调 Activity 的 onContentChanged 方法通知 Activity 视图已经发生改变
        cb.onContentChanged();
    }
}
```

这个时候 DecorView 还没有被 WindowManager 正式添加。在 ActivityThread 的 handleResumeActivity 方法中，首先会调用 Activity 的 onResume 方法，接着调用 Activity 的 makeVisible()，完成 DecorView 的添加和显示过程：

``Activity.java``

```java
void makeVisible() {
    if (!mWindowAdded) {
        ViewManager wm = getWindowManager();
        wm.addView(mDecor, getWindow().getAttributes());
        mWindowAdded = true;
    }
    mDecor.setVisibility(View.VISIBLE);
}
```

### Dialog 的 Window 创建过程

Dialog 的 Window 的创建过程和 Activity 类似，创建同样是通过 PolicyManager 的 makeNewWindow 方法完成的，创建后的对象实际就是 PhoneWindow。当 Dialog 被关闭时，会通过 WindowManager 来移除 DecorView：mWindowManager.removeViewImmediate(mDecor)。

``Dialog.java``

```java
Dialog(@NonNull Context context, @StyleRes int themeResId, boolean      createContextThemeWrapper) {
    ···
    mWindowManager = (WindowManager) context.getSystemService(Context.WINDOW_SERVICE);

    final Window w = new PhoneWindow(mContext);
    mWindow = w;
    w.setCallback(this);
    w.setOnWindowDismissedCallback(this);
    w.setOnWindowSwipeDismissedCallback(() -> {
        if (mCancelable) {
            cancel();
        }
    });
    w.setWindowManager(mWindowManager, null, null);
    w.setGravity(Gravity.CENTER);

    mListenersHandler = new ListenersHandler(this);
}
```

普通 Dialog 必须采用 Activity 的 Context，采用 Application 的 Context 就会报错，是因为应用 token 所导致，应用 token 一般只有 Activity 拥有。系统 Window 比较特殊，不需要 token。

### Toast 的 Window 创建过程

Toast 属于系统 Window ，由于其具有定时取消功能，所以系统采用了 Handler。Toast 的内部有两类 IPC 过程，第一类是 Toast 访问 NotificationManagerService，第二类是 NotificationManagerService 回调 Toast 里的 TN 接口。

Toast 内部的视图由两种方式，一种是系统默认的样式，另一种是 setView 指定一个自定义 View，它们都对应 Toast 的一个内部成员 mNextView。

``Toast.java``

```java
public void show() {
    if (mNextView == null) {
        throw new RuntimeException("setView must have been called");
    }

    INotificationManager service = getService();
    String pkg = mContext.getOpPackageName();
    TN tn = mTN;
    tn.mNextView = mNextView;

    try {
        service.enqueueToast(pkg, tn, mDuration);
    } catch (RemoteException e) {
        // Empty
    }
}
···

public void cancel() {
    mTN.cancel();
}

```

``NotificationManagerService.java``

```java
private void showNextToastLocked() {
    ToastRecord record = mToastQueue.get(0);
    while (record != null) {
        if (DBG) Slog.d(TAG, "Show pkg=" + record.pkg + " callback=" + record.callback);
        try {
            record.callback.show();
            scheduleTimeoutLocked(record, false);
            return;
        } catch (RemoteException e) {
            Slog.w(TAG, "Object died trying to show notification " + record.callback
                    + " in package " + record.pkg);
            // remove it from the list and let the process die
            int index = mToastQueue.indexOf(record);
            if (index >= 0) {
                mToastQueue.remove(index);
            }
            keepProcessAliveLocked(record.pid);
            if (mToastQueue.size() > 0) {
                record = mToastQueue.get(0);
            } else {
                record = null;
            }
        }
    }
}

···
private void scheduleTimeoutLocked(ToastRecord r, boolean immediate)
{
    Message m = Message.obtain(mHandler, MESSAGE_TIMEOUT, r);
    long delay = immediate ? 0 : (r.duration == Toast.LENGTH_LONG ? LONG_DELAY : SHORT_DELAY);
    mHandler.removeCallbacksAndMessages(r);
    mHandler.sendMessageDelayed(m, delay);
}
```

## 怎么在Service中创建Dialog对话框？

1.在我们取得Dialog对象后，需给它设置类型，即：

```
dialog.getWindow().setType(WindowManager.LayoutParams.TYPE_SYSTEM_ALERT)
```

2.在Manifest中加上权限:

```
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINOW" />
```

