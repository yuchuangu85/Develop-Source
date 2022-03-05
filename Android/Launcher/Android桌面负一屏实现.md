# Android桌面负一屏实现
负一屏的实现主流有两种方式

Launcher自行开发，往Workspace中插入一个自定义的CellLayout来作为负一屏的容器。 这种方式是最常用的方案。
利用Google的feed屏方案，基于ILauncherOverlay和ILauncherOverlayCallback这两个接口来实现，Launcher作为客户端，负一屏是一个独立应用作为服务端，通过aidl来通信来实现加载负一屏以及支持相互滑页。
方案一： 桌面自行开发，可以快速完美的加入负一屏，但是当负一屏的业务越来越多时，会拖累launcher的性能，内存等。
实现可以关注我的下一篇博文 [Launcher负一屏实现第一章](https://blog.csdn.net/u013032242/article/details/83036747)

要解决方案一的问题，我们就需要考虑将负一屏独立出去，作为单独的进程运行，这种实现我们需要思考两个问题。

问题1：负一屏作为独立进程，以什么方式显示出来？

问题2：负一屏的界面属于什么层级，和launcher是什么关系，如何交互？

本文主要讲述下第二种方案的实现。

客户端Launcher的实现

首先我们导入google官方定义了两个interface接口，ILauncherOverlay和ILauncherOverlayCallback。
ILauncherOverlay定义的方法是客户端调用服务端的，方法的实现在service端，ILauncherOverlayCallback是服务端回调客户端的，实现在客户端。

```java
public interface ILauncherOverlay extends IInterface {
    void closeOverlay(int options) throws RemoteException;
    void endScroll() throws RemoteException;
    String getVoiceSearchLanguage() throws RemoteException;
    boolean isVoiceDetectionRunning() throws RemoteException;
    void onPause() throws RemoteException;
    void onResume() throws RemoteException;
    void onScroll(float progress) throws RemoteException;
    void openOverlay(int options) throws RemoteException;
    void requestVoiceDetection(boolean start) throws RemoteException
    void startScroll() throws RemoteException;
    void windowAttached(WindowManager.LayoutParams attrs, ILauncherOverlayCallback callbacks, int options) throws RemoteException;
    void windowDetached(boolean isChangingConfigurations) throws    RemoteException;
}
public interface ILauncherOverlayCallback extends IInterface {
    void overlayScrollChanged(float progress) throws RemoteException;
    void overlayStatusChanged(int status) throws RemoteException;
}
```

Launcher通过bindservice后，拿到ILauncherOverlay实体，来和服务端进行通信，传递关键信息给服务端。

Launcher在onAttachedToWindow的时候，会通过windowAttached(WindowManager.LayoutParams attrs, ILauncherOverlayCallback callbacks, int options)方法，将Launcher的Activity的window属性，和launcher实例化出来的ILauncherOverlayCallback.Stub回调传给服务端。

```java
public final void onAttachedToWindow() {
    if (!this.mDestroyed) {
       this.setWindowAttrs(this.mActivity.getWindow().getAttributes());
    }
}
```
针对用户在屏幕的滑动操作, 调用startScroll（）, onScroll(float progress), endScroll()来通知服务端;

launcher这边打开关闭负一屏，用openOverlay（）,closeOverlay（）来通知服务端；

launcher作为客户端，相应的改动并不大，绑定服务后，只需要在正确的地方调用接口，来通知服务端，重点和难点在服务端的实现。

服务端Service的实现

首先新建一个工程，将客户端的ILauncherOverlay和ILauncherOverlayCallback接口文件拷贝过来，注意目录要保持一致（aidl通信）。
AndroidManifest中定义个Service，实例化ILauncherOverlay.Stub，并通过onBind来return给客户端；

```java
ILauncherOverlay.Stub mStub = new ILauncherOverlay.Stub() {
  @Override
 public IBinder asBinder() {
     return this;
 }
 @Override
 public void startScroll() throws RemoteException {
     Log.i(TAG, "CustomPageService startScroll");
     Message.obtain(CustomPageService.this.mainThreadHandler, >>OverlayCallback.START_SCROLL).sendToTarget();
  }

 @Override
     public void onScroll(final float scroll) throws RemoteException {
     Message.obtain(CustomPageService.this.mainThreadHandler, OverlayCallback.UPDATE_SCROLL,  scroll).sendToTarget();
  }

 @Override
     public void endScroll() throws RemoteException {
     Log.i(TAG, "CustomPageService endScroll");
     Message.obtain(CustomPageService.this.mainThreadHandler, OverlayCallback.END_SCROLL).sendToTarget();
  }

 @Override
     public void windowAttached(WindowManager.LayoutParams layoutParams, ILauncherOverlayCallback >>callback, int options) throws RemoteException {
     Log.i(TAG, "CustomPageService windowAttached attr="+layoutParams.toString());
     Bundle bundle = new Bundle();
     bundle.putParcelable("layout_params", layoutParams);
     bundle.putInt("client_options", options);
     OverlayCallback overlayCallback = new OverlayCallback(CustomPageService.this);
     mainThreadHandler = new Handler(Looper.getMainLooper(), overlayCallback);
     Message.obtain(CustomPageService.this.mainThreadHandler, OverlayCallback.WINDOW_ATTACHED, >>Pair.create(bundle, callback)).sendToTarget();
   }

 @Override
     public void windowDetached(boolean isChangingConfigurations) throws RemoteException {
     Log.i(TAG, "CustomPageService windowDetached");
     Message.obtain(CustomPageService.this.mainThreadHandler,  >>OverlayCallback.WINDOW_DETACHCHED, isChangingConfigurations).sendToTarget();
   }

 @Override
     public void closeOverlay(int var1) throws RemoteException {
     Log.i(TAG, "CustomPageService closeOverlay");
     Message.obtain(CustomPageService.this.mainThreadHandler, >>OverlayCallback.CLOSE_OVERLY).sendToTarget();

 @Override
     public void onPause() throws RemoteException {
     Log.i(TAG, "CustomPageService onPause");
  }

 @Override
     public void onResume() throws RemoteException {
     Log.i(TAG, "CustomPageService onResume");
  }

 @Override
     public void openOverlay(int var1) throws RemoteException {
     Log.i(TAG, "CustomPageService openOverlay");
     Message.obtain(CustomPageService.this.mainThreadHandler, >>OverlayCallback.OPEN_OVERLY).sendToTarget();
  }

 @Override
     public void requestVoiceDetection(boolean var1) throws RemoteException {
     Log.i(TAG, "CustomPageService requestVoiceDetection");
 }

 @Override
     public String getVoiceSearchLanguage() throws RemoteException {
     return null;
 }

 @Override
     public boolean isVoiceDetectionRunning() throws RemoteException {
     return false;
   }
};
```

在客户端调用服务端的windowAttached的时候，服务端的主要实现如下
需要开始创建一个新的window,，然后将客户端的LayoutParams中的部分属性赋值给服务端的window

```java
Dialog dialog = new Dialog(context, dialogTheme);
this.window = dialog.getWindow();
window.setWindowManager(null, layoutParams.token,
new ComponentName(this, this.getBaseContext().getClass()).flattenToShortString(), true);
windowManager = window.getWindowManager();
mWindowShift = 0;
container = new FrameLayout(this);
layoutParams.width = WindowManager.LayoutParams.MATCH_PARENT;
layoutParams.height = WindowManager.LayoutParams.MATCH_PARENT;
layoutParams.flags |= WindowManager.LayoutParams.FLAG_LAYOUT_IN_SCREEN
| WindowManager.LayoutParams.FLAG_LAYOUT_NO_LIMITS;
layoutParams.gravity = Gravity.LEFT;
layoutParams.type = WindowManager.LayoutParams.TYPE_DRAWN_APPLICATION;
layoutParams.softInputMode = WindowManager.LayoutParams.SOFT_INPUT_ADJUST_RESIZE;
window.setAttributes(layoutParams);
window.setContentView(container);
windowView = window.getDecorView();
windowManager.addView(windowView, window.getAttributes());
setVisible(false);
callback.overlayStatusChanged(1);
```

到此，就可以在launcher的上层显示一个负一屏的window了，如果不给负一屏容器设置一个translationX的话，默认该window是盖在launcher上的，我们可以将translationX默认值设置在屏幕以外。

之后，我们可以根据launcher调用startScroll（），onScroll（float progress），endScroll(), 传过来的scroll值，来改变负一屏window中的view的translationX，来达到负一屏滑动的效果，同时将scroll的progress通过ILauncherOverlayCallback中的overlayScrollChanged（float progress）回调给launcher，launcher可以让workspace做相应的translationX，这样就可以达到负一屏和launcher的滑动关联，而不是负一屏盖在launcher上。

记得在滑页结束后，来确定哪个window需要focus，通过设置window的flag来完成focus控制。

总结：

第一种常规方案，能快速实现负一屏，但有一些缺点，如运行在launcher进程中，对launcher是有消耗的，当负一屏的业务越来越多时，会对launcher内存，性能造成影响。

第二种方案的实现，得以让负一屏独立出去，负一屏的显示依赖launcher的activity的token，但是运行却在独立的进程中。
————————————————
版权声明：本文为CSDN博主「世界末日不是谣言」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u013032242/article/details/82885110