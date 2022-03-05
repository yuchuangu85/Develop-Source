<h1 align="center">Android 10 中的隐私权变更</h1>

文档地址：[https://developer.android.google.cn/about/versions/10/privacy](https://developer.android.google.cn/about/versions/10/privacy)

## 重大变更

* 外部存储访问权限范围限定为应用文件和媒体？
  为了让用户更好地管理自己的文件并减少混乱，以 Android 10（API 级别 29）及更高版本为目标平台的应用在默认情况下被赋予了对外部存储设备的分区访问权限（即分区存储）。此类应用只能看到本应用专有的目录（通过 Context.getExternalFilesDir() 访问）以及特定类型的媒体。除非您的应用需要访问存放在应用的专有目录以及 MediaStore 之外的文件，否则最好使用分区存储。
  
  | 文件位置                     | 所需权限                                          | 访问方法                          | 卸载应用时是否移除文件？ |
  | ---------------------------- | ------------------------------------------------- | --------------------------------- | ------------------------ |
  | 特定于应用的目录             | 无                                                | getExternalFilesDir()             | 是                       |
  | 媒体集合（照片、视频、音频） | READ_EXTERNAL_STORAGE（仅当访问其他应用的文件时） | MediaStore                        | 否                       |
  | Documents and other files    | 无                                                | 存储访问框架（DocumentsProvider） | 否                       |

*您可以使用<u>存储访问框架</u>访问上表中显示的每一个位置，而无需请求任何权限

使用分区存储的应用对自己创建的文件始终拥有读/写权限，无论文件是否位于应用的专有目录内。因此，如果您的应用仅保存和访问自己创建的文件，则无需请求获得 <u>READ_EXTERNAL_STORAGE</u> 或 <u>WRITE_EXTERNAL_STORAGE</u> 权限。

不过，若要访问其他应用创建的文件，则必须满足以下两个条件：

您的应用已获得 >READ_EXTERNAL_STORAGE 权限。

这些文件位于以下其中一个明确定义的媒体集合中：

* 照片：存储在 <u>MediaStore.Images</u> 中。
* 视频：存储在 <u>MediaStore.Video</u> 中。
* 音频文件：存储在 <u>MediaStore.Audio</u> 中。

为了访问其他应用创建的任何其他文件，您的应用必须使用存储访问框架，该框架允许用户选择特定文件。

在您的应用完全兼容分区存储之前，您可以根据应用的目标 SDK 级别或 requestLegacyExternalStorage 清单属性，暂时选择停用分区存储：

```xml
<manifest ... >    
    <application android:requestLegacyExternalStorage="true" ... >        ...     
    </application>
</manifest>
```

警告：明年，主要平台版本将要求所有应用都使用分区存储，无论应用的目标 SDK 级别是多少。因此，您应该提前确保您的应用能够使用分区存储。为此，请确保针对搭载 Android 10（API 级别 29）及更高版本的设备启用了该行为。

* 在后台运行时访问设备位置信息需要权限
  Android 10 引入了 <u>ACCESS_BACKGROUND_LOCATION</u> 权限。

除非符合以下条件之一，否则应用将被视为在后台访问位置信息：

属于该应用的 Activity 可见。
该应用运行的某个前台设备已声明<u>前台服务类型</u>为 location。

* 针对从后台启动 Activity 的限制

几乎在所有情况下，后台应用都应<u>显示有时效性的通知</u>，以便向用户提供紧急信息，而非直接启动 Activity。此类通知的适用情形包括处理来电或正在响铃的闹钟。

在特定情况下，您的应用可能需要立即引起用户的注意，例如闹钟正在响铃或有来电时。您以前可能已出于此目的配置了应用，也就是当应用在后台运行时启动 Activity。

要在运行 Android 10（API 级别 29）或更高版本的设备上提供类似的行为，请完成本指南中描述的步骤。

* 创建高优先级通知

  ```java
  Intent fullScreenIntent = new Intent(this, CallActivity.class);    
  PendingIntent fullScreenPendingIntent = PendingIntent.getActivity(this, 0,            fullScreenIntent, PendingIntent.FLAG_UPDATE_CURRENT);
  
  NotificationCompat.Builder notificationBuilder = new NotificationCompat.Builder(this, CHANNEL_ID) 
  .setSmallIcon(R.drawable.notification_icon)
  .setContentTitle("Incoming call")
  .setContentText("(919) 555-1234")
  .setPriority(NotificationCompat.PRIORITY_HIGH)        .setCategory(NotificationCompat.CATEGORY_CALL)
  .setFullScreenIntent(fullScreenPendingIntent, true);
  
  Notification incomingCallNotification = notificationBuilder.build();
  ```

* 向用户显示通知

```java
startForeground(notificationId, notification);
```



####  标识符和数据

* 移除了联系人亲密程度信息
  	从 Android 10 开始，平台不再跟踪联系人亲密程度信息。因此，如果您的应用对用户的联系人进行搜索，系统将不会按互动频率对搜索结果排序。

有关 ContactsProvider 的指南包含一项描述特定字段和方法的声明（从 Android 10 开始，这些字段和方法在所有设备上已作废）。

* 随机分配 MAC 地址

* 对 /proc/net 文件系统的访问权限实施了限制
  在搭载 Android 10 或更高版本的设备上，应用无法访问 /proc/net，其中包含与设备的网络状态相关的信息。需要访问这些信息的应用（如VPN）应使用 NetworkStatsManager 或 ConnectivityManager 类。

* 对不可重置的设备标识符实施了限制

  

  受影响的方法包括：

```java
Build
getSerial()
TelephonyManager
getImei()
getDeviceId()
getMeid()
getSimSerialNumber()
getSubscriberId()
```



* 限制了对剪贴板数据的访问权限

  除非您的应用是默认输入法 (IME) 或是目前处于焦点的应用，否则它无法访问 Android 10 或更高版本平台上的剪贴板数据。

* 保护 USB 设备序列号
  如果您的应用以 Android 10 或更高版本为目标平台，则该应用只能在用户授予其访问 USB 设备或配件的权限后才能读取序列号。

####  摄像头和连接性

* 对访问摄像头详情和元数据的权限实施了限制
  Android 10 更改了 getCameraCharacteristics() 方法默认返回的信息的广度。具体而言，您的应用必须具有 CAMERA 权限才能访问此方法的返回值中可能包含的设备特定元数据。

* 对启用和停用 WLAN 实施了限制
  以 Android 10 或更高版本为目标平台的应用无法启用或停用 WLAN。WifiManager.setWifiEnabled() 方法始终返回 false。

  如果您需要提示用户启用或停用 WLAN，请使用设置面板。

* 对直接访问已配置的 WLAN 网络实施了限制
  为了保护用户隐私，只有系统应用和设备政策控制器 (DPC) 支持手动配置WLAN 网络列表。给定 DPC可以是设备所有者，也可以是资料所有者。

  

  如果应用以 Android 10 或更高版本为目标平台，并且应用不是系统应用或 DPC，则下列方法不会返回有用数据：

  * getConfiguredNetworks() 方法始终返回空列表。

    注意：如果运营商应用调用 getConfiguredNetworks()，则系统返回的列表仅包含运营商配置的网络。

  * 每个返回整数值的网络操作方法（addNetwork() 和 updateNetwork()）始终返回 -1。

  * 每个返回布尔值的网络操作（removeNetwork()、reassociate()、enableNetwork()、disableNetwork()、reconnect() 和 disconnect()）始终返回 false。

如果您的应用需要连接到 WLAN 网络，请使用以下备用方法：

* 要触发与 WLAN 网络的即时本地连接，请在标准 NetworkRequest 对象中使用 WifiNetworkSpecifier。

* *要添加 WLAN 网络以便考虑为用户提供互联网访问权限，请使用 WifiNetworkSuggestion 对象。您可以分别通过调用 addNetworkSuggestions() 和 removeNetworkSuggestions() 来添加和移除显示在自动连接网络选择对话框中的网络。这些方法不需要任何位置权限。

  

* 一些电话 API、蓝牙 API 和WLAN API需要精确位置权限
  如果应用以 Android 10 或更高版本为目标平台，则它必须具有 ACCESS_FINE_LOCATION 权限才能使用 WLAN、WLAN 感知或蓝牙API中的一些方法。以下部分列举了受影响的类和方法。

电话

 * TelephonyManager
   * getCellLocation()
   * getAllCellInfo()
   * requestNetworkScan()
   * requestCellInfoUpdate()
   * getAvailableNetworks()
   * getServiceState()

* TelephonyScanManager
  * requestNetworkScan()
* TelephonyScanManager.NetworkScanCallback
  * onResults()
* PhoneStateListener
  * onCellLocationChanged()
  * onCellInfoChanged()
  * onServiceStateChanged()

WLAN

* WifiManager
  * startScan()
  * getScanResults()
  * getConnectionInfo()
  * getConfiguredNetworks()

* WifiAwareManager
* WifiP2pManager
* WifiRttManager

蓝牙

* BluetoothAdapter
  * startDiscovery()
  * startLeScan()

* BluetoothAdapter.LeScanCallback

* BluetoothLeScanner
  * startScan()

####  权限

* 限制对屏幕内容的访问
  为了保护用户的屏幕内容，Android 10 更改了 READ_FRAME_BUFFER、CAPTURE_VIDEO_OUTPUT 和 CAPTURE_SECURE_VIDEO_OUTPUT 权限的作用域，从而禁止以静默方式访问设备的屏幕内容。从 Android 10 开始，这些权限只能通过签名访问。

  需要访问设备屏幕内容的应用应使用 MediaProjection API，此 API 会显示提示，要求用户同意访问

* 从界面中移除了权限组
  从 Android 10 开始，应用无法在界面中查询权限的分组方式。

#### 限制非 SDK 接口

为了帮助确保应用的稳定性和兼容性，Android 平台开始限制应用在 Android 9（API 级别 28）中使用非 SDK 接口。Android 10 包含更新后的受限制非 SDK 接口列表（基于与 Android 开发者之间的协作以及最新的内部测试）。我们的目标是在限制使用非 SDK 接口之前确保有可用的公开替代方案。

如果您不打算以 Android 10（API 级别 29）为目标平台，那么其中一些变更可能不会立即对您产生影响。虽然您目前仍然可以使用灰名单中的一些非 SDK 接口（取决于您的应用的目标 API 级别），但如果您使用任何非 SDK 方法或字段，则应用无法运行的风险终归较高。

如果您不确定自己的应用是否使用了非 SDK 接口，则可以测试该应用，进行确认。如果您的应用依赖于非 SDK 接口，则应该开始计划迁移到 SDK 替代方案。不过，我们知道某些应用具有使用非 SDK 接口的有效用例。如果您无法为应用中的某项功能找到使用非 SDK 接口的替代方案，则应该请求新的公共 API。

要了解详情，请参阅 Android 10 中有关限制非 SDK 接口的更新以及针对非 SDK 接口的限制。

#### WLAN 直连广播

  在 Android 10 中，以下与 WLAN 直连相关的广播不具有粘性：

* WIFI_P2P_CONNECTION_CHANGED_ACTION

* WIFI_P2P_THIS_DEVICE_CHANGED_ACTION

  

   如果您的应用依赖于在注册时接收这些广播（因为其之前一直具有粘性），请在初始化时使用适当的 get() 方法获取信息。

#### WLAN 感知功能

  Android 10 扩大了支持范围，现在可以使用WLAN感知数据路径轻松创建 TCP/UDP 套接字。要创建连接到 ServerSocket 的TCP/UDP套接字，客户端设备需要知道服务器的IPv6地址和端口。这在之前需要通过频外方式进行通信（例如使用 BT 或 WLAN 感知第 2 层消息传递），或者使用其他协议（例如mDNS）通过频内方式发现。而借助 Android 10，可以将此类消息作为网络设置的一部分进行传递。

  服务器可以执行以下任一操作：

 * 初始化 ServerSocket 并设置或获取要使用的端口。
 * 将端口信息指定为WLAN感知网络请求的一部分

### HTTPS连接变更

  如果在 Android 10 上运行的应用将 null 传递给 setSSLSocketFactory()，则会出现 IllegalArgumentException。在以前的版本中，将 null 传递给 setSSLSocketFactory() 与传入当前的默认 SSL 套接字工厂效果相同。

#### android.preference 库已弃用

  从 Android 10 开始，将弃用 android.preference 库。开发者应该改为使用 AndroidX preference 库，这是 Android Jetpack 的一部分。如需获取其他有助于迁移和开发的资源，请查看经过更新的设置指南以及我们的公开示例应用和参考文档。

#### Android Beam 已弃用

  在 Android 10 中，我们正式弃用了 Android Beam，这是一项旧版功能，可通过近距离无线通信 (NFC) 在多个设备之间启动数据共享。我们还弃用了一些相关的 NFC API。Android Beam 仍可供需要的设备制造商合作伙伴使用，但它已不再处于积极的开发阶段。不过，Android 仍将继续支持其他的 NFC 功能和 API，并且从标签和付款中读取数据等使用场景仍将继续按预期执行。

#### 创新技术和新体验

* 可折叠设备
* 5G 网络
* 通知中的智能回复
* 深色主题
* 手势导航
* 设置面板
