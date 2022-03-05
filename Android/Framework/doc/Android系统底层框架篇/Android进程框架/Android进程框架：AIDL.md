<h1 align="center">Android系统应用框架篇：AIDL</h1>

在介绍AIDL的原理之前先写一个简单的Demo。

**举例**

1 定义一个AIDL文件

```java
package com.guoxiaoxing.android.framework.demo.system.aidl;

// Declare any non-default types here with import statements

interface IMyAidlInterface {
    /**
     * Demonstrates some basic types that you can use as parameters
     * and return values in AIDL.
     */
    void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat,
            double aDouble, String aString);

    String getName();
}

```

rebuild一下，它会自动生成一个Java接口，它内部会生成一个抽象类Stub，它本质上是一个Binder，它可以用来进行跨进程通信。如下所示：

```java
public interface IMyAidlInterface extends android.os.IInterface
{

//Stub类实现，它继承于Binder，同样也实现了IMyAidlInterface接口，读取Proxy传递过来的参数，并写入返回给Proxy的值。
/** Local-side IPC implementation stub class. */
public static abstract class Stub extends android.os.Binder implements com.guoxiaoxing.android.framework.demo.system.aidl.IMyAidlInterface
{

//接口的包名
private static final java.lang.String DESCRIPTOR = "com.guoxiaoxing.android.framework.demo.system.aidl.IMyAidlInterface";
/** Construct the stub at attach it to the interface. */
public Stub()
{
this.attachInterface(this, DESCRIPTOR);
}
/**
 * Proxy类本身是私有的，为了防止外部对它进行修改，通过该方法返回Proxy对象，供外部调用。
 * 
 * Cast an IBinder object into an com.guoxiaoxing.android.framework.demo.system.aidl.IMyAidlInterface interface,
 * generating a proxy if needed.
 */
public static com.guoxiaoxing.android.framework.demo.system.aidl.IMyAidlInterface asInterface(android.os.IBinder obj)
{
if ((obj==null)) {
return null;
}
android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
if (((iin!=null)&&(iin instanceof com.guoxiaoxing.android.framework.demo.system.aidl.IMyAidlInterface))) {
return ((com.guoxiaoxing.android.framework.demo.system.aidl.IMyAidlInterface)iin);
}
return new com.guoxiaoxing.android.framework.demo.system.aidl.IMyAidlInterface.Stub.Proxy(obj);
}
@Override public android.os.IBinder asBinder()
{
return this;
}
@Override public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException
{
switch (code)
{
case INTERFACE_TRANSACTION:
{
reply.writeString(DESCRIPTOR);
return true;
}
case TRANSACTION_basicTypes:
{
data.enforceInterface(DESCRIPTOR);
int _arg0;
_arg0 = data.readInt();
long _arg1;
_arg1 = data.readLong();
boolean _arg2;
_arg2 = (0!=data.readInt());
float _arg3;
_arg3 = data.readFloat();
double _arg4;
_arg4 = data.readDouble();
java.lang.String _arg5;
_arg5 = data.readString();
this.basicTypes(_arg0, _arg1, _arg2, _arg3, _arg4, _arg5);
reply.writeNoException();
return true;
}
case TRANSACTION_getName:
{
data.enforceInterface(DESCRIPTOR);
java.lang.String _result = this.getName();
reply.writeNoException();
reply.writeString(_result);
return true;
}
}
return super.onTransact(code, data, reply, flags);
}

//Proxy类，它实现了我们定义的IMyAidlInterface接口，写入传递给Stub的参数，读取Stub返回的值。
private static class Proxy implements com.guoxiaoxing.android.framework.demo.system.aidl.IMyAidlInterface
{
private android.os.IBinder mRemote;
//构造方法中传入远程Binder
Proxy(android.os.IBinder remote)
{
mRemote = remote;
}
//返回远程Binder
@Override public android.os.IBinder asBinder()
{
return mRemote;
}
public java.lang.String getInterfaceDescriptor()
{
return DESCRIPTOR;
}
/**
     * Demonstrates some basic types that you can use as parameters
     * and return values in AIDL.
     */
@Override public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, java.lang.String aString) throws android.os.RemoteException
{
android.os.Parcel _data = android.os.Parcel.obtain();
android.os.Parcel _reply = android.os.Parcel.obtain();
try {
_data.writeInterfaceToken(DESCRIPTOR);
_data.writeInt(anInt);
_data.writeLong(aLong);
_data.writeInt(((aBoolean)?(1):(0)));
_data.writeFloat(aFloat);
_data.writeDouble(aDouble);
_data.writeString(aString);
mRemote.transact(Stub.TRANSACTION_basicTypes, _data, _reply, 0);
_reply.readException();
}
finally {
_reply.recycle();
_data.recycle();
}
}
@Override public java.lang.String getName() throws android.os.RemoteException
{
android.os.Parcel _data = android.os.Parcel.obtain();
android.os.Parcel _reply = android.os.Parcel.obtain();
java.lang.String _result;
try {
_data.writeInterfaceToken(DESCRIPTOR);
mRemote.transact(Stub.TRANSACTION_getName, _data, _reply, 0);
_reply.readException();
_result = _reply.readString();
}
finally {
_reply.recycle();
_data.recycle();
}
return _result;
}
}
static final int TRANSACTION_basicTypes = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
static final int TRANSACTION_getName = (android.os.IBinder.FIRST_CALL_TRANSACTION + 1);
}
/**
     * Demonstrates some basic types that you can use as parameters
     * and return values in AIDL.
     */
public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, java.lang.String aString) throws android.os.RemoteException;
public java.lang.String getName() throws android.os.RemoteException;

```

2 定义一个Service组件，我们给它指定一个新的process属性，让它运行在新的进程中（因为我们要用AIDL做跨进程调用），然后
将IMyAidlInterface.Stub指定给Service组件。

```java
public class AidlService extends Service {
    public AidlService() {
    }

    @Override
    public IBinder onBind(Intent intent) {
        return new AidlBinder();
    }

    private class AidlBinder extends IMyAidlInterface.Stub{

        @Override
        public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, String aString) throws RemoteException {

        }

        @Override
        public String getName() throws RemoteException {
            return "hello AIDL";
        }
    }
}
```

如下图所示。存在两个进程。

<img src="../../../art/native/aidl_service.png"/>

3 定义一个Activity组件，绑定AidlService，并调用它的远程方法。

```java
public class AidlActivity extends AppCompatActivity {

    private IMyAidlInterface iMyAidlInterface;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_aidl);

        Intent intent = new Intent(AidlActivity.this, AidlService.class);
        bindService(intent, new ServiceConnection() {
            @Override
            public void onServiceConnected(ComponentName name, IBinder service) {
                iMyAidlInterface = IMyAidlInterface.Stub.asInterface(service);

            }

            @Override
            public void onServiceDisconnected(ComponentName name) {

            }
        }, Context.BIND_AUTO_CREATE);


        findViewById(R.id.btn_message).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                try {
                    Toast.makeText(v.getContext(), iMyAidlInterface.getName(), Toast.LENGTH_SHORT).show();
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
                ;
            }
        });
    }
}
```

以上就是AIDL调用的大致流程。

好了，我们针对上面的流程来总结一下AIDL。

Android 接口定义语言 (AIDL) 与您可能使用过的其他接口语言 (IDL) 类似。您可以利用它定义客户端与服务均认可的编程接口，以便二者使用进程间通信 (IPC) 进行相互通信。在 Android 中，一个进程通常无法访问另一个进程的内存。因此，为进行通信，进程需将其对象分解成可供操作系统理解的原语，并将其编组为可供您操作的对象。编写执行该编组操作的代码较为繁琐，因此 Android 会使用 AIDL 为您处理此问题。

**注意：**只有在需要不同应用的客户端通过 IPC 方式访问服务，并且希望在服务中进行多线程处理时，您才有必要使用 AIDL。如果您无需跨不同应用执行并发 IPC，则应通过[实现 Binder](https://developer.android.com/guide/components/bound-services?hl=zh-cn#Binder) 来创建接口；或者，如果您想执行 IPC，但*不*需要处理多线程，请[使用 Messenger ](https://developer.android.com/guide/components/bound-services?hl=zh-cn#Messenger)来实现接口。无论如何，在实现 AIDL 之前，请您务必理解[绑定服务](https://developer.android.com/guide/components/bound-services?hl=zh-cn)。

在开始设计 AIDL 接口之前，请注意，AIDL 接口的调用是直接函数调用。您无需对发生调用的线程做任何假设。实际情况的差异取决于调用是来自本地进程中的线程，还是远程进程中的线程。具体而言：

- 来自本地进程的调用在发起调用的同一线程内执行。如果该线程是您的主界面线程，则其将继续在 AIDL 接口中执行。如果该线程是其他线程，则其便是在服务中执行代码的线程。因此，只有在本地线程访问服务时，您才能完全控制哪些线程在服务中执行（但若出现此情况，您根本无需使用 AIDL，而应通过[实现 Binder 类](https://developer.android.com/guide/components/bound-services?hl=zh-cn#Binder)来创建接口）。
- 远程进程的调用分派自线程池，且平台会在您自己的进程内部维护该线程池。您必须为来自未知线程，且多次调用同时发生的传入调用做好准备。换言之，AIDL 接口的实现必须基于完全的线程安全。如果调用来自同一远程对象上的某个线程，则该调用将**依次**抵达接收器端。
- `oneway` 关键字用于修改远程调用的行为。使用此关键字后，远程调用不会屏蔽，而只是发送事务数据并立即返回。最终接收该数据时，接口的实现会将其视为来自 `Binder` 线程池的常规调用（普通的远程调用）。如果 `oneway` 用于本地调用，则不会有任何影响，且调用仍为同步调用。

## 定义 AIDL 接口

您必须在 `.aidl` 文件中使用 Java 编程语言的语法定义 AIDL 接口，然后将其保存至应用的源代码（在 `src/` 目录中）内，这类应用会托管服务或与服务进行绑定。

在构建每个包含 `.aidl` 文件的应用时，Android SDK 工具会生成基于该 `.aidl` 文件的 `IBinder` 接口，并将其保存到项目的 `gen/` 目录中。服务必须视情况实现 `IBinder` 接口。然后，客户端应用便可绑定到该服务，并调用 `IBinder` 中的方法来执行 IPC。

如要使用 AIDL 创建绑定服务，请执行以下步骤：

1. 创建 .aidl 文件

   此文件定义带有方法签名的编程接口。

2. 实现接口

   Android SDK 工具会基于您的 `.aidl` 文件，使用 Java 编程语言生成接口。此接口拥有一个名为 `Stub` 的内部抽象类，用于扩展 `Binder` 类并实现 AIDL 接口中的方法。您必须扩展 `Stub` 类并实现这些方法。

3. 向客户端公开接口

   实现 `Service` 并重写 `onBind()`，从而返回 `Stub` 类的实现。

**注意：**如果您在首次发布 AIDL 接口后对其进行更改，则每次更改必须保持向后兼容性，以免中断其他应用使用您的服务。换言之，由于只有在将您的 `.aidl` 文件复制到其他应用后，才能使其访问服务接口，因而您必须保留对原始接口的支持。

### 1. 创建 .aidl 文件

AIDL 使用一种简单语法，允许您通过一个或多个方法（可接收参数和返回值）来声明接口。参数和返回值可为任意类型，甚至是 AIDL 生成的其他接口。

您必须使用 Java 编程语言构建 `.aidl` 文件。每个 `.aidl` 文件均须定义单个接口，并且只需要接口声明和方法签名。

默认情况下，AIDL 支持下列数据类型：

- Java 编程语言中的所有原语类型（如 `int`、`long`、`char`、`boolean` 等）

- `String`

- `CharSequence`

- ```
   List
   ```

   `List` 中的所有元素必须是以上列表中支持的数据类型，或者您所声明的由 AIDL 生成的其他接口或 Parcelable 类型。您可选择将 `List` 用作“泛型”类（例如，`List<String>`）。尽管生成的方法旨在使用 `List` 接口，但另一方实际接收的具体类始终是 `ArrayList`。

- ```
   Map
   ```

   `Map` 中的所有元素必须是以上列表中支持的数据类型，或者您所声明的由 AIDL 生成的其他接口或 Parcelable 类型。不支持泛型 Map（如 `Map<String,Integer>` 形式的 Map）。尽管生成的方法旨在使用 `Map` 接口，但另一方实际接收的具体类始终是 `HashMap`。

即使您在与接口相同的包内定义上方未列出的附加类型，亦须为其各自加入一条 `import` 语句。

定义服务接口时，请注意：

- 方法可带零个或多个参数，返回值或空值。

- 所有非原语参数均需要指示数据走向的方向标记。这类标记可以是

    

   ```
   in
   ```

   或

   

   ```
   out 
   ```

   或

    

   ```
   inout
   ```

   （见下方示例）。

   原语默认为 `in`，不能是其他方向。

   **注意：**您应将方向限定为真正需要的方向，因为编组参数的开销较大。

- 生成的 `IBinder` 接口内包含 `.aidl` 文件中的所有代码注释（import 和 package 语句之前的注释除外）。

- 您可以在 ADL 接口中定义 String 常量和 int 字符串常量。例如：`const int VERSION = 1;`。

- 方法调用由 [transact() 代码](https://developer.android.com/reference/android/os/IBinder?hl=zh-cn#transact(int, android.os.Parcel, android.os.Parcel, int))分派，该代码通常基于接口中的方法索引。由于这会增加版本控制的难度，因此您可以向方法手动配置事务代码：`void method() = 10;`。

- 使用 `@nullable` 注释可空参数或返回类型。

以下是 `.aidl` 文件示例：

```
// IRemoteService.aidl
package com.example.android

// Declare any non-default types here with import statements
/** Example service interface */
internal interface IRemoteService {
    /** Request the process ID of this service, to do evil things with it. */
    val pid:Int

    /** Demonstrates some basic types that you can use as parameters
     * and return values in AIDL.
     */
    fun basicTypes(anInt:Int, aLong:Long, aBoolean:Boolean, aFloat:Float,
                 aDouble:Double, aString:String)
}
```

您只需将 `.aidl` 文件保存至项目的 `src/` 目录内，这样在构建应用时，SDK 工具便会在项目的 `gen/` 目录中生成 `IBinder` 接口文件。生成文件的名称与 `.aidl` 文件的名称保持一致，区别在于其使用 `.java` 扩展名（例如，`IRemoteService.aidl` 生成的文件名是 `IRemoteService.java`）。

如果您使用 Android Studio，增量构建几乎会立即生成 Binder 类。如果您不使用 Android Studio，则 Gradle 工具会在您下一次开发应用时生成 Binder 类。因此，在编写完 `.aidl` 文件后，您应立即使用 `gradle assembleDebug`（或 `gradle assembleRelease`）构建项目，以便您的代码能够链接到生成的类。

### 2. 实现接口

当您构建应用时，Android SDK 工具会生成以 `.aidl` 文件命名的 `.java` 接口文件。生成的接口包含一个名为 `Stub` 的子类（例如，`YourInterface.Stub`），该子类是其父接口的抽象实现，并且会声明 `.aidl` 文件中的所有方法。

**注意：**`Stub` 还会定义几个辅助方法，其中最值得注意的是 `asInterface()`，该方法会接收 `IBinder`（通常是传递给客户端 `onServiceConnected()` 回调方法的参数），并返回 Stub 接口的实例。如需了解如何进行此转换的更多详情，请参阅[调用 IPC 方法](https://developer.android.com/guide/components/aidl?hl=zh-cn#Calling)部分。

如要实现 `.aidl` 生成的接口，请扩展生成的 `Binder` 接口（例如，`YourInterface.Stub`），并实现继承自 `.aidl` 文件的方法。

以下示例展示使用匿名实例实现 `IRemoteService` 接口（由以上 `IRemoteService.aidl` 示例定义）的过程：

```java
private final IRemoteService.Stub binder = new IRemoteService.Stub() {
    public int getPid(){
        return Process.myPid();
    }
    public void basicTypes(int anInt, long aLong, boolean aBoolean,
        float aFloat, double aDouble, String aString) {
        // Does nothing
    }
};
```

现在，`binder` 是 `Stub` 类的一个实例（一个 `Binder`），其定义了服务的远程过程调用 (RPC) 接口。在下一步中，我们会向客户端公开此实例，以便客户端能与服务进行交互。

在实现 AIDL 接口时，您应注意遵守以下规则：

- 由于无法保证在主线程上执行传入调用，因此您一开始便需做好多线程处理的准备，并对您的服务进行适当构建，使其达到线程安全的标准。
- 默认情况下，RPC 调用是同步调用。如果您知道服务完成请求的时间不止几毫秒，则不应从 Activity 的主线程调用该服务，因为这可能会使应用挂起（Android 可能会显示“Application is Not Responding”对话框）— 通常，您应从客户端内的单独线程调用服务。
- 您引发的任何异常都不会回传给调用方。

### 3. 向客户端公开接口

在为服务实现接口后，您需要向客户端公开该接口，以便客户端进行绑定。如要为您的服务公开该接口，请扩展 `Service` 并实现 `onBind()`，从而返回实现生成的 `Stub` 的类实例（如前文所述）。以下是向客户端公开 `IRemoteService` 示例接口的服务示例。

```java
public class RemoteService extends Service {
    @Override
    public void onCreate() {
        super.onCreate();
    }

    @Override
    public IBinder onBind(Intent intent) {
        // Return the interface
        return binder;
    }

    private final IRemoteService.Stub binder = new IRemoteService.Stub() {
        public int getPid(){
            return Process.myPid();
        }
        public void basicTypes(int anInt, long aLong, boolean aBoolean,
            float aFloat, double aDouble, String aString) {
            // Does nothing
        }
    };
}
```

现在，当客户端（如 Activity）调用 `bindService()` 以连接此服务时，客户端的 `onServiceConnected()` 回调会接收服务的 `onBind()` 方法所返回的 `binder` 实例。

客户端还必须拥有接口类的访问权限，因此如果客户端和服务在不同应用内，则客户端应用的 `src/` 目录内必须包含 `.aidl` 文件（该文件会生成 `android.os.Binder` 接口，进而为客户端提供 AIDL 方法的访问权限）的副本。

当客户端在 `onServiceConnected()` 回调中收到 `IBinder` 时，它必须调用 `*YourServiceInterface*.Stub.asInterface(service)`，以将返回的参数转换成 `*YourServiceInterface*` 类型。例如：

```java
IRemoteService iRemoteService;
private ServiceConnection mConnection = new ServiceConnection() {
    // Called when the connection with the service is established
    public void onServiceConnected(ComponentName className, IBinder service) {
        // Following the example above for an AIDL interface,
        // this gets an instance of the IRemoteInterface, which we can use to call on the service
        iRemoteService = IRemoteService.Stub.asInterface(service);
    }

    // Called when the connection with the service disconnects unexpectedly
    public void onServiceDisconnected(ComponentName className) {
        Log.e(TAG, "Service has unexpectedly disconnected");
        iRemoteService = null;
    }
};
```

如需查看更多示例代码，请参阅 [ApiDemos](https://android.googlesource.com/platform/development/+/master/samples/ApiDemos) 中的 [`RemoteService.java`](https://android.googlesource.com/platform/development/+/master/samples/ApiDemos/src/com/example/android/apis/app/RemoteService.java) 类。

## 通过 IPC 传递对象

您可以通过 IPC 接口，将某个类从一个进程发送至另一个进程。不过，您必须确保 IPC 通道的另一端可使用该类的代码，并且该类必须支持 `Parcelable` 接口。支持 `Parcelable` 接口很重要，因为 Android 系统能通过该接口将对象分解成可编组至各进程的原语。

如要创建支持 `Parcelable` 协议的类，您必须执行以下操作：

1. 让您的类实现 `Parcelable` 接口。

2. 实现 `writeToParcel`，它会获取对象的当前状态并将其写入 `Parcel`。

3. 为您的类添加 `CREATOR` 静态字段，该字段是实现 `Parcelable.Creator` 接口的对象。

4. 最后，创建声明 Parcelable 类的

    

   ```
   .aidl
   ```

    

   文件（遵照下文

    

   ```
   Rect.aidl
   ```

    

   文件所示步骤）。

   如果您使用的是自定义编译进程，*请勿*在您的构建中添加 `.aidl` 文件。此 `.aidl` 文件与 C 语言中的头文件类似，并未经过编译。

AIDL 会在其生成的代码中使用这些方法和字段，以对您的对象进行编组和解编。

例如，下方的 `Rect.aidl` 文件可创建 Parcelable 类型的 `Rect` 类：

```
package android.graphics;

// Declare Rect so AIDL can find it and knows that it implements
// the parcelable protocol.
parcelable Rect;
```

以下示例展示 `Rect` 类如何实现 `Parcelable` 协议。

```java
import android.os.Parcel;
import android.os.Parcelable;

public final class Rect implements Parcelable {
    public int left;
    public int top;
    public int right;
    public int bottom;

    public static final Parcelable.Creator<Rect> CREATOR = new Parcelable.Creator<Rect>() {
        public Rect createFromParcel(Parcel in) {
            return new Rect(in);
        }

        public Rect[] newArray(int size) {
            return new Rect[size];
        }
    };

    public Rect() {
    }

    private Rect(Parcel in) {
        readFromParcel(in);
    }

    public void writeToParcel(Parcel out, int flags) {
        out.writeInt(left);
        out.writeInt(top);
        out.writeInt(right);
        out.writeInt(bottom);
    }

    public void readFromParcel(Parcel in) {
        left = in.readInt();
        top = in.readInt();
        right = in.readInt();
        bottom = in.readInt();
    }

    public int describeContents() {
        return 0;
    }
}
```

`Rect` 类中的编组相当简单。请查看 `Parcel` 的其他相关方法，了解您可以向 Parcel 写入哪些其他类型的值。

**警告：**请勿忘记从其他进程中接收数据的安全问题。在本例中，`Rect` 从 `Parcel` 读取四个数字，但您需确保：无论调用方目的为何，这些数字均在可接受的值范围内。如需详细了解如何防止应用受到恶意软件侵害、保证应用安全，请参阅[安全与权限](https://developer.android.com/guide/topics/security/security?hl=zh-cn)。

## 带软件包参数（包含 Parcelable 类型）的方法

如果您的 AIDL 接口包含接收软件包作为参数（预计包含 Parcelable 类型）的方法，则在尝试从软件包读取之前，请务必通过调用 `Bundle.setClassLoader(ClassLoader)` 设置软件包的类加载器。否则，即使您在应用中正确定义 Parcelable 类型，也会遇到 `ClassNotFoundException`。例如，

如果您有 `.aidl` 文件：

```java
// IRectInsideBundle.aidl
package com.example.android;

/** Example service interface */
interface IRectInsideBundle {
    /** Rect parcelable is stored in the bundle with key "rect" */
    void saveRect(in Bundle bundle);
}
```

如下方实现所示，在读取 `Rect` 之前，`ClassLoader` 已在 `Bundle` 中完成显式设置

```java
private final IRectInsideBundle.Stub binder = new IRectInsideBundle.Stub() {
    public void saveRect(Bundle bundle){
        bundle.setClassLoader(getClass().getClassLoader());
        Rect rect = bundle.getParcelable("rect");
        process(rect); // Do more with the parcelable.
    }
};
```

## 调用 IPC 方法

如要调用通过 AIDL 定义的远程接口，调用类必须执行以下步骤：

1. 在项目的 `src/` 目录中加入 `.aidl` 文件。
2. 声明一个 `IBinder` 接口实例（基于 AIDL 生成）。
3. 实现 `ServiceConnection`。
4. 调用 `Context.bindService()`，从而传入您的 `ServiceConnection` 实现。
5. 在 `onServiceConnected()` 实现中，您将收到一个 `IBinder` 实例（名为 `service`）。调用 `*YourInterfaceName*.Stub.asInterface((IBinder)*service*)`，以将返回的参数转换为 *YourInterface* 类型。
6. 调用您在接口上定义的方法。您应始终捕获 `DeadObjectException` 异常，系统会在连接中断时抛出此异常。您还应捕获 `SecurityException` 异常，当 IPC 方法调用中两个进程的 AIDL 定义发生冲突时，系统会抛出此异常。
7. 如要断开连接，请使用您的接口实例调用 `Context.unbindService()`。

有关调用 IPC 服务的几点说明：

- 对象是跨进程计数的引用。
- 您可以方法参数的形式发送匿名对象。

如需了解有关绑定服务的详细信息，请阅读[绑定服务](https://developer.android.com/guide/components/bound-services?hl=zh-cn#Binding)文档。

以下示例代码摘自 ApiDemos 项目的远程服务示例，展示如何调用 AIDL 创建的服务。



