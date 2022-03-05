<h1 align="center">ContentProvider</h1>

[toc]

ContentProvider 管理对结构化数据集的访问。它们封装数据，并提供用于定义数据安全性的机制。 内容提供程序是连接一个进程中的数据与另一个进程中运行的代码的标准界面。

ContentProvider 无法被用户感知，对于一个 ContentProvider 组件来说，它的内部需要实现增删该查这四种操作，它的内部维持着一份数据集合，这个数据集合既可以是数据库实现，也可以是其他任何类型，如 List 和 Map，内部的 insert、delete、update、query 方法需要处理好线程同步，因为这几个方法是在 Binder 线程池中被调用的。

ContentProvider 通过 Binder 向其他组件乃至其他应用提供数据。当 ContentProvider 所在的进程启动时，ContentProvider 会同时启动并发布到 AMS 中，需要注意的是，这个时候 ContentProvider 的 onCreate 要先于 Application 的 onCreate 而执行。

## 基本使用

```java
// Queries the user dictionary and returns results
mCursor = getContentResolver().query(
    UserDictionary.Words.CONTENT_URI,   // The content URI of the words table
    mProjection,                        // The columns to return for each row
    mSelectionClause                    // Selection criteria
    mSelectionArgs,                     // Selection criteria
    mSortOrder);                        // The sort order for the returned rows
```

```java
public class Installer extends ContentProvider {

    @Override
    public boolean onCreate() {
        return true;
    }

    @Nullable
    @Override
    public Cursor query(@NonNull Uri uri, @Nullable String[] projection, @Nullable String selection, @Nullable String[] selectionArgs, @Nullable String sortOrder) {
        return null;
    }

    @Nullable
    @Override
    public String getType(@NonNull Uri uri) {
        return null;
    }

    @Nullable
    @Override
    public Uri insert(@NonNull Uri uri, @Nullable ContentValues values) {
        return null;
    }

    @Override
    public int delete(@NonNull Uri uri, @Nullable String selection, @Nullable String[] selectionArgs) {
        return 0;
    }

    @Override
    public int update(@NonNull Uri uri, @Nullable ContentValues values, @Nullable String selection, @Nullable String[] selectionArgs) {
        return 0;
    }
}
```

> ContentProvider 和 sql 在实现上有什么区别?
>
> - ContentProvider 屏蔽了数据存储的细节，内部实现透明化，用户只需关心 uri 即可(是否匹配)
> - ContentProvider 能实现不同 app 的数据共享，sql 只能是自己程序才能访问
> - Contentprovider 还能增删本地的文件,xml等信息

## ContentProvider的权限管理(读写分离，权限控制-精确到表级，URL控制)。

　对于ContentProvider暴露出来的数据，应该是存储在自己应用内存中的数据，对于一些存储在外部存储器上的数据，并不能限制访问权限，使用ContentProvider就没有意义了。对于ContentProvider而言，有很多权限控制，可以在AndroidManifest.xml文件中对节点的属性进行配置，一般使用如下一些属性设置：

- android:grantUriPermssions         		临时许可标志。
- android:permission                         Provider 读写权限。
- android:readPermission                     Provider的读权限。
- android:writePermission                    Provider的写权限。
- android:enabled                            标记允许系统启动Provider。
- android:exported                           标记允许其他应用程序使用这个Provider。
- android:multiProcess                       标记允许系统启动Provider相同的进程中调用客户端。

## 说说ContentProvider、ContentResolver、ContentObserver 之间的关系？

ContentProvider：管理数据，提供数据的增删改查操作，数据源可以是数据库、文件、XML、网络等，ContentProvider为这些数据的访问提供了统一的接口，可以用来做进程间数据共享。

ContentResolver：ContentResolver可以为不同URI操作不同的ContentProvider中的数据，外部进程可以通过ContentResolver与ContentProvider进行交互。

ContentObserver：观察ContentProvider中的数据变化，并将变化通知给外界。