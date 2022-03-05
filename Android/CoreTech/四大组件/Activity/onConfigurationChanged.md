<h1 align="center">onConfigurationChanged</h1>

**newConfig**：新的设备配置信息当系统的配置信息发生改变时，系统会调用此方法。注意，只有在配置文件

AndroidManifest 中处理了 configChanges 属性 对应的设备配置，该方法才会被调用。如果发生设备配置与在配置文件中设置的不一致，则 **Activity**会被销毁并 **使用新的配置重建**。 

例如：当屏幕方向发生改变时，Activity 会被销毁重建，如果在 AndroidManifest 文件中处理屏幕方向配置信息如下：

```xml
<activity 
	android:name=".MainActivity"
	android:label="@string/app_name"
	android:configchanges="oriention|screenSize"/>
```

Activity 不会被销毁重建，而是调用 onConfigurationChanged 方法。如果 configChanges 只设置了 orientation，则当其他设备配置信息改变时， Activity 依然会销毁重建，且不会调用 onConfigurationChanged。 

例如，在上面的配置的情况下，如果语言改变了，Acitivyt 就会销毁重建，且不会调用 onConfigurationChanged 方法。

**configChanges** **设置取值**

| Value                | Description                                                  |
| -------------------- | ------------------------------------------------------------ |
| “mcc“                | The IMSI mobile country code (MCC) has changed — that is, a SIM hasbeen detected and updated the MCC.移动国家号码，由三位数字组成，每个国家都有自己独立的MCC，可以识别手机用户所属国家。 |
| “mnc“                | The IMSI mobile network code (MNC) has changed — that is, a SIM hasbeen detected and updated the MNC.移动网号，在一个国家或者地区中，用于区分手机用户的服务商。 |
| “locale“             | The locale has changed — for example, the user has selected a new language that text should be displayed in.用户所在地区发生变化。 |
| “touchscreen“        | The touchscreen has changed. (This should never normally happen.) |
| “keyboard“           | The keyboard type has changed — for example, the user has plugged in an external keyboard.键盘模式发生变化，例如：用户接入外部键盘输入。 |
| “keyboardHidden“     | The keyboard accessibility has changed — for example, the user has slid the keyboard out to expose it.用户打开手机硬件键盘 |
| “navigation“         | The navigation type has changed. (This should never normally happen.) |
| “orientation“        | The screen orientation has changed — that is, the user has rotated the device.设备旋转，横向显示和竖向显示模式切换。 |
| “fontScale“          | The font scaling factor has changed — that is, the user has selected a new global font size.全局字体大小缩放发生改变 |
| “screenLayout”       | 屏幕布局发生了变化-用户选择了新的全局字号                    |
| “uiMode”             | 用户界面模式发生了变化-这可能是用户将设备放入桌面、车载座或者夜间模式发生变化所致， |
| “screenSize”         | 当前可用屏幕尺寸发生变化。它表示当前可用尺寸相对于当前纵横比的变化，因此会在用户在横向与纵向之间切换时发生变化。不过，如果你的应用面向API级别12或者更低级别，则Activity始终会自行处理此配置变更（即便是在Android3.2或者更高版本的设备上运行，此配置变更也不会重启Activity） |
| “smallestScreenSize” | 屋里屏幕尺寸发生变化。它表示与方向无关的尺寸变化，因此只有在实际屋里屏幕尺寸发生变化（如切换到外部显示器）时才变化。对此配置的变更对应于smallestWidth配置变化。不过，如果你的 应用面向API级别12或者更低级别，则Activity始终会自行处理此配置变化（即便是在Android3.2或者更高版本的设备上运行，次配置变更也不会重新启动Activity） |
| “layoutDirection”    | 布局方向发生变化。例如：从左至右（LTR）更改从右至左（RTL）。此项为API17中新增配置。 |

**注意：**横竖屏切换的属性是 orientation。如果 targetSdkVersion 的值大于等于13，则如下配置才会回调 onConfigurationChanged 方法：

 ```xml
android:configChanges="orientation|screenSize"
 ```

 如果 targetSdkVersion 的值小于 13，则只要配置：

 ```xml
android:configChanges="orientation"
 ```

 网上有很多文章写说横竖屏切换时 onConfigurationChanged 方法 没有调用，使

用如下的配置：

 ```xml
android:configChanges="orientation|keyboard|keyboardHidden"
 ```

 但是，其实查官方文档，只要配置 android:configChanges="orientation|screenSize" 就可以了。 

**扩展：**当用户接入一个外设键盘时，默认软键盘会自动隐藏，系统自动使用外设键盘。这个过程 Activity 的销毁和隐藏执行了两次。并且 onConfigurationChanged() 不会调用。但是在配置文件中设置 **android:configChanges="keyboardHidden|keyboard"**。当接入外设键盘或者拔出外设键盘时，调用的周期是先调用onConfigurationChanged()周期后销毁重建。

 **在这里有一个疑点，为什么有两次的销毁重建？** 

其中一次的销毁重建可以肯定是因为外设键盘的插入和拔出。当设置 android:configChanges="keyboardHidden|keyboard" 之后，就不会销毁重建，而是调用 onConfigurationChanged()方法。

 **但是还有一次销毁重建一直存在。**

 经过测试，当接入外设键盘时，除了键盘类型的改变，触摸屏也发生了变化。因为使用外设键盘，触摸屏不能使用了。（如果是接入触摸板，不知道会不会有这个问题？欢迎大家提供意见)。这里，我接入的是键盘，所以触摸屏不能使用了。

**总结：如果是键盘类型发生了改变，则** **configChanges** **属性 配置如下** **Activity** **才不会销毁重建，且回调** **onConfigurationChanged** **方法：**

```xml
<activity 
	android:name=".MainActivity"
	android:label="@string/app_name"
	android:configchanges="keyboard|keyboardHidden/touchscreen"/>
```

**note:**这里的外置物理键盘可以是游戏手柄、扫描枪、键盘等等。

 官方文档：

 https://developer.android.com/guide/topics/manifest/activity-element.html 小楠篇

 在手机 APP 开发的时候，一般默认会适配竖屏，游戏开发除外。但是在 Android 平板电脑开发中，屏幕旋转的问题比较突出，可以这样说，平板电脑的最初用意就是横屏使用的，比较方便，用户会经常旋转我们设备的屏幕。

 **屏幕旋转的适配问题以及遇到的一些坑**

 http://www.jianshu.com/p/19393bb08e4f

上面我的文章中提到了一些坑，包括 View 的测量不准确， onConfigurationChanged 的回调不确定，今天主要分析一下 onConfigurationChanged 调用的不确定性因素。

 关于这个问题，笔者在网上搜索了一下关于为什么 onConfigurationChanged 的方法不会被调用，基本都是说清单文件里面没有正确配置，因为在 Android2.3 以后需要增加 **screenSize** 这个配置。

 但是完全搜索不到关于我的问题的搜索结果，毕竟做 Android 平板的并不多，因此写下来记录自己的学习过程。

 **关于官方文档**

 我们知道，在 Activity、View（ViewGroup）、Fragment、Service、Content Provider 等等在设备的配置发生变化的时候，会回调 onConfigurationChanged 的方法。实质上主要是 Activity 中收到 AMS 的通知，回调，然后把事件分发到 Window、Fragment、ActionBar 等。

 下面我们可以通过 Activity 的 onConfigurationChanged 方法 源码可以看到：

```java
    public void onConfigurationChanged(@NonNull Configuration newConfig) {
        if (DEBUG_LIFECYCLE) Slog.v(TAG, "onConfigurationChanged " + this + ": " + newConfig);
        mCalled = true;

        mFragments.dispatchConfigurationChanged(newConfig);

        if (mWindow != null) {
            // Pass the configuration changed event to the window
            mWindow.onConfigurationChanged(newConfig);
        }

        if (mActionBar != null) {
            // Do this last; the action bar will need to access
            // view changes from above.
            mActionBar.onConfigurationChanged(newConfig);
        }
    }
```

 这里我们讨论的是为什么当我们的界面在设备配置发生变化的时候（屏幕旋转），有时候并不会回调 onConfigurationChanged 呢？

关于 Activity 的官方文档有下面一句话：

Called by the system when the device configuration changes while your activity is running. Note that this will *only* be called if you have selected configurations you would like to handle with the `R.attr.configChanges` attribute in your manifest. If any configuration change occurs that is not selected to be reported by that attribute, then instead of reporting it the system will stop and restart the activity (to have it launched with the new configuration).

At the time that this function has been called, your Resources object will have been updated to return resource values matching the new configuration.

也就是说，在设备配置发生变化的时候，会回调 onConfigurationChanged，但是前提条件是当你的 Activity（组件）还在运行的时候。

 这就很明显了，说明一旦你的界面暂停以后就不会回调这个方法了。但是这样会导致一个问题，就是你的界面跳转到其他界面的时候（当前界面暂停），然后发生了一次屏幕旋转，再返回的时候，你的界面虽然旋转了，但是并没有回调 onConfigurationChanged 方法，并没有执行你的 UI 适配代码。

**源码分析**

 想到四大组件，我们第一时间应该会想到 AMS(ActivityManagerService)，没错，今天我们的始发站就是 AMS。

AMS 里面搜索了一下关键字 Configuration，发现了 updateConfigurationLocked 这个方法（没有说明的情况下，都是只给出省略版）：

相信眼尖的朋友一定会看出来，在这里由 AMS 创建了 Configuration 对象，然后通过进程间通信，通知我们的 app 进程。

```java
   @Override
    public boolean updateConfiguration(Configuration values) {
        mAmInternal.enforceCallingPermission(CHANGE_CONFIGURATION, "updateConfiguration()");

        synchronized (mGlobalLock) {
            if (mWindowManager == null) {
                Slog.w(TAG, "Skip updateConfiguration because mWindowManager isn't set");
                return false;
            }

            if (values == null) {
                // sentinel: fetch the current configuration from the window manager
                values = mWindowManager.computeNewConfiguration(DEFAULT_DISPLAY);
            }

            mH.sendMessage(PooledLambda.obtainMessage(
                    ActivityManagerInternal::updateOomLevelsForDisplay, mAmInternal,
                    DEFAULT_DISPLAY));

            final long origId = Binder.clearCallingIdentity();
            try {
                if (values != null) {
                    Settings.System.clearConfiguration(values);
                }
                updateConfigurationLocked(values, null, false, false /* persistent */,
                        UserHandle.USER_NULL, false /* deferResume */,
                        mTmpUpdateConfigurationResult);
                return mTmpUpdateConfigurationResult.changes != 0;
            } finally {
                Binder.restoreCallingIdentity(origId);
            }
        }
    }
```

