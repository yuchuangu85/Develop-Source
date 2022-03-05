# Launcher3集成谷歌负一屏

# 概览

负一屏是一个特殊的页面，同时桌面设置中有开关控制是否显示负一屏。负一屏的实现有两种方案：一是在桌面中新增一个页面定制负一屏的内容；二是单独开发一个负一屏的apk。两个应用通过aidl通信来实现负一屏加载以及相互滑页。现在基本都采用第二种方式来实现自己的负一屏。

## Launcher新增一页显示负一屏

这种负一屏的实现方式是通过在Launcher主屏左侧增加一个Celllayout页面来显示负一屏的内容，（参见Lanucher3源码中addToCustomContentPage方法），并通过hasCustomContentToLeft方法来控制自定义的负一屏是否显示。（Android P之后的Launcher3源码已经删除了这部分代码）所有的定制内容都写在这个新增的Celllayout页面上，但是随着负一屏的内容越来越多这种实现方式会明显拖累Launcher的内存和性能，而Launcher对内存和性能要求又很高，甚至在谷歌的GTS测试中有对应的测试项，我想这也是为什么谷歌在Android P之后删除这部分逻辑的原因。

## 负一屏作为一个单独的应用进行集成

负一屏作为一个单独应用集成进系统是目前最优的负一屏实现方案，这样两个应用（Launcher和负一屏）就通过AIDL进行通信实现滑动和加载逻辑。这时Launcher相当于一个客户端应用，而负一屏应用相当于一个服务端应用，客户端应用通过接口调用服务端方法实现相关加载和控制逻辑。下面探索下如何在Launcher3中实现加载谷歌的负一屏应用。谷歌释放的GMS包中提供了一个示例应用（SearchLauncher）来集成谷歌的负一屏。大致分为以下几步：

### 1. 导入并编译launcher_client.jar

两个不同的应用通过AIDL进行通信，而谷歌负一屏应用将这部分逻辑封装进了launcher_client.jar方便我们调用。launcher_client.jar反编译之后代码基本已经无法阅读，不过可以看出主要是通过bindService来和作为服务端的负一屏应用进行通信,其中最主要的就是LauncherClient.java这个类，Launcher就是通过这个类来向负一屏传递信息。因此首先在Launcher中Android.mk中加入如下代码：



```go
include $(CLEAR_VARS)
LOCAL_MODULE := lib_launcherClient
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE_CLASS := JAVA_LIBRARIES
LOCAL_SRC_FILES := libs/launcher_client.jar
LOCAL_UNINSTALLABLE_MODULE := true
LOCAL_SDK_VERSION := current
include $(BUILD_PREBUILT)
```

然后引用该静态Java库



```go
LOCAL_STATIC_JAVA_LIBRARIES := lib_launcherClient
```

### 2. 新添加OverlayCallbackImpl.java和SearchLauncherCallbacks.java两个接口实现类

这两个新添加的类在谷歌释放的GMS包中的示例应用中可以找到，这两个接口实现类类似于MVP模式中的P层的作用。主要是将Launcher的相关信息传递到服务端也就是负一屏应用，而后负一屏应用根据这些信息执行相应的操作。

- OverlayCallbackImpl.java实现了Launcher3中的LauncherOverlay接口(接口定义如下所示)和launcher_client.jar中的LauncherClientCallbacks，从这个接口实现类可以看出Launcher把自己的滑动相关信息传递了出去，当Launcher调用这些接口时，负一屏应用即开始调用startMove，endMove，updateMove开始滑动。



```csharp
    public interface LauncherOverlay {

        /**
         * Touch interaction leading to overscroll has begun
         */
        void onScrollInteractionBegin();

        /**
         * Touch interaction related to overscroll has ended
         */
        void onScrollInteractionEnd();

        /**
         * Scroll progress, between 0 and 100, when the user scrolls beyond the leftmost
         * screen (or in the case of RTL, the rightmost screen).
         */
        void onScrollChange(float progress, boolean rtl);

        /**
         * Called when the launcher is ready to use the overlay
         * @param callbacks A set of callbacks provided by Launcher in relation to the overlay
         */
        void setOverlayCallbacks(LauncherOverlayCallbacks callbacks);
    }
```



```java
public interface LauncherClientCallbacks {
    void onOverlayScrollChanged(float progress);

    void onServiceStateChanged(boolean overlayAttached, boolean hotwordActive);
}
```

- SearchLauncherCallbacks.java主要实现了Launcher3中的LauncherCallbacks,其次实现了两个监听OnSharedPreferenceChangeListener,OnDeviceProfileChangeListener，从这个接口实现类可以看出，Launcher将自己的生命周期方法也传递给了负一屏，以便负一屏执行相同的生命周期如onCreate，onResume，onStart，onStop，onDestroy，onAttachedToWindow等，这样保证了两个应用的生命周期一致性。其中Launcher在onAttachedToWindow的时候，会通过windowAttached方法，将Launcher的Activity的window属性回调传给服务端，服务端开始创建一个新的window，然后将客户端的LayoutParams中的部分属性赋值给服务端的window，这样就可以在launcher的上层显示一个负一屏的window了，如果不给负一屏容器设置一个translationX的话，默认该window是盖在launcher上的，我们可以将translationX默认值设置在屏幕以外。之后可以根据launcher调用滑动方法传过来的scroll值，来改变负一屏window中的view的translationX，来达到负一屏滑动的效果，同时通过onOverlayScrollChanged（float progress）将滑动progress回调给launcher，launcher可以让workspace做相应的translationX，这样就可以达到负一屏和launcher的滑动关联，而不是负一屏盖在launcher上。



```java
public interface LauncherCallbacks {

    /*
     * Activity life-cycle methods. These methods are triggered after
     * the code in the corresponding Launcher method is executed.
     */
    void onCreate(Bundle savedInstanceState);
    void onResume();
    void onStart();
    void onStop();
    void onPause();
    void onDestroy();
    void onSaveInstanceState(Bundle outState);
    void onActivityResult(int requestCode, int resultCode, Intent data);
    void onRequestPermissionsResult(int requestCode, String[] permissions,
            int[] grantResults);
    void onAttachedToWindow();
    void onDetachedFromWindow();
    void dump(String prefix, FileDescriptor fd, PrintWriter w, String[] args);
    void onHomeIntent(boolean internalStateHandled);
    boolean handleBackPressed();
    void onTrimMemory(int level);

    /*
     * Extension points for providing custom behavior on certain user interactions.
     */
    void onLauncherProviderChange();
    void bindAllApplications(ArrayList<AppInfo> apps);

    /**
     * Starts a search with {@param initialQuery}. Return false if search was not started.
     */
    boolean startSearch(
            String initialQuery, boolean selectInitialQuery, Bundle appSearchData);

    /*
     * Extensions points for adding / replacing some other aspects of the Launcher experience.
     */
    boolean hasSettings();
}
```

### 3. 最后在Launcher.java中开启负一屏，在onCreate中添加如下代码



```cpp
mLauncherCallbacks = new SearchLauncherCallbacks(this);
setLauncherCallbacks(mLauncherCallbacks);
```

因为在Launcher3源码中已经存在如下代码，实际上是已经预留好了负一屏的接口。



```csharp
if (mLauncherCallbacks != null) {
    mLauncherCallbacks.onCreate(savedInstanceState);
}
```

# 总结

若是要集成其他三方负一屏则需要其他三方负一屏的接口，若是要开发自己的负一屏可以参照Google的这个负一屏方案和代码进行开发。要深入了解其中的原理还需要阅读launcher_client.jar中关于AIDL通信的代码和其他逻辑，OverlayCallbackImpl.java和SearchLauncherCallbacks.java两个接口实现类中的逻辑也要仔细阅读，这两类当起了一个桥梁的作用的，由于本人知识有限如有不对请谅解，并欢迎补充。



作者：一袋小乞丐
链接：https://www.jianshu.com/p/b07ff2c44a6c
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。