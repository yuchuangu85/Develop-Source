<h1 align="center">Android Resource</h1>

## Layout

### 1. What is the difference between @+id/ and @id/?

- android:id="**@android:id**/myView"

   Android系统内置的资源，还有：`@android:color/`, `@android:drawable/`, `@android:layout/`,…；例如：android.R.layout.simple_list_item_1，@android:color/transpa等。

- android:id=”**@+id**/myView”

   带有"+"表明正在声明一个新的资源。

- android:id="**@id**/myView"

   表明已经存在了myView这个资源。

