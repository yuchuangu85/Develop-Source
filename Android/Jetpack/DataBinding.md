<h1 align="center">DataBinding</h1>

[toc]

## 配置

```
 android {
 	...
    dataBinding {
        enabled = true
    }
}
```

## 用法

### 基础运算符

DataBinding 支持在布局文件中使用以下运算符、表达式和关键字

- 算术 `+ - / * %`
- 字符串合并`+`
- 逻辑`&& ||`
- 二元`& | ^`
- 一元 `+ - ! ~`
- 移位`>> >>> <<`
- 比较`== > < >= <=`
- `Instanceof`
- `Grouping ()`
- `character, String, numeric, null`
- `Cast`
- 方法调用
- `Field` 访问
- `Array`访问 []
- 三元`?:`

目前不支持以下操作

- `this`
- `super`
- `new`
- 显示泛型调用

### Type

| Type              | Normal reference | Expression reference |
| :---------------- | :--------------- | :------------------- |
| String[]          | @array           | @stringArray         |
| int[]             | @array           | @intArray            |
| TypedArray        | @array           | @typedArray          |
| Animator          | @animator        | @animator            |
| StateListAnimator | @animator        | @stateListAnimator   |
| color int         | @color           | @color               |
| ColorStateList    | @color           | @colorStateList      |

### Null Coalescing

空合并运算符 **`??`** 会取第一个不为 `null` 的值作为返回值

```xml
<TextView
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="@{user.name ?? user.password}" />
```

等价于

```xml
android:text="@{user.name != null ? user.name : user.password}"
```

### 属性控制

可以通过变量值来控制 View 的属性

```xml
<TextView
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="可见性变化"
    android:visibility="@{user.male  ? View.VISIBLE : View.GONE}" />
```

### 避免空指针异常

DataBinding 也会自动帮助我们避免空指针异常
例如，如果 **“@{userInfo.password}”** 中 **userInfo** 为 **null** 的话，**userInfo.password** 会被赋值为默认值 **null**，而不会抛出空指针异常

### BindingAdapter

dataBinding 提供了 **BindingAdapter** 这个注解用于支持自定义属性，或者是修改原有属性。注解值可以是已有的 xml 属性，例如 `android:src`、`android:text`等，也可以自定义属性然后在 xml 中使用

例如，对于一个 ImageView ，我们希望在某个变量值发生变化时，可以动态改变显示的图片，此时就可以通过 BindingAdapter 来实现

需要先定义一个静态方法，为之添加 BindingAdapter 注解，注解值是为 ImageView 控件自定义的属性名，而该静态方法的两个参数可以这样来理解：当 ImageView 控件的 url 属性值发生变化时，dataBinding 就会将 ImageView 实例以及新的 url 值传递给 loadImage() 方法，从而可以在此动态改变 ImageView 的相关属性

```java
@BindingAdapter({"url"})
public static void loadImage(ImageView view, String url) {
	Log.d(TAG, "loadImage url : " + url);
}
```

在 xml 文件中关联变量值，当中，bind 这个名称可以自定义

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:bind="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">
    
    <data>
        <import type="com.github.ixiaow.databindingsample.model.Image" />
        <variable
            name="image"
            type="Image" />
    </data>
    
    <android.support.constraint.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <ImageView
            android:id="@+id/image"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:src="@drawable/ic_launcher_background"
            bind:url="@{image.url}" />
    </android.support.constraint.ConstraintLayout>
</layout>
```

BindingAdapter 更为强大的一点是可以覆盖 Android 原先的控件属性。例如，可以设定每一个 Button 的文本都要加上后缀：“-Button”

```java
@BindingAdapter("android:text")
public static void setText(Button view, String text) {
	view.setText(text + "-Button");
}
```

```xml
<Button
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:onClick="@{()->handler.onClick(image)}"
    android:text='@{"改变图片Url"}'/>
```

这样，整个工程中使用到了 **“android:text”** 这个属性的控件，其显示的文本就会多出一个后缀

### BindingConversion

dataBinding 还支持对数据进行转换，或者进行类型转换

与 BindingAdapter 类似，以下方法会将布局文件中所有以`@{String}`方式引用到的`String`类型变量加上后缀`-conversionString`

```java
@BindingConversion
public static String conversionString(String text) {
	return text + "-conversionString";
}
```

xml 文件

```xml
<TextView
	android:layout_width="match_parent"
	android:layout_height="wrap_content"
	android:text='@{"xxx"}'
	android:textAllCaps="false"/>
```

可以看到，对于 Button 来说，BindingAdapter 和 BindingConversion 同时生效了，而 BindingConversion 的优先级要高些, 此外，BindingConversion 也可以用于转换属性值的类型

看以下布局，此处在向 `background` 和 `textColor` 两个属性赋值时，直接就使用了字符串，按正常情况来说这自然是会报错的，但有了 BindingConversion 后就可以自动将字符串类型的值转为需要的 `Drawable` 和 `Color` 了

```xml
<TextView
	android:layout_width="match_parent"
	android:layout_height="wrap_content"
	android:background='@{"红色"}'
	android:padding="20dp"
	android:text="红色背景蓝色字"
	android:textColor='@{"蓝色"}'/>

<TextView
	android:layout_width="match_parent"
	android:layout_height="wrap_content"
	android:layout_marginTop="20dp"
	android:background='@{"蓝色"}'
	android:padding="20dp"
	android:text="蓝色背景红色字"
	android:textColor='@{"红色"}'/>

```

```java

@BindingConversion
public static Drawable convertStringToDrawable(String str) {
    if (str.equals("红色")) {
        return new ColorDrawable(Color.parseColor("#FF4081"));
    }

    if (str.equals("蓝色")) {
        return new ColorDrawable(Color.parseColor("#3F51B5"));
    }
    return new ColorDrawable(Color.parseColor("#344567"));
}

@BindingConversion
public static int convertStringToColor(String str) {
    if (str.equals("红色")) {
        return Color.parseColor("#FF4081");
    }

    if (str.equals("蓝色")) {
        return Color.parseColor("#3F51B5");
    }
    return Color.parseColor("#344567");
}
```

### 事件处理

- 事件处理分为两种：方法引用和监听器绑定；
- 方法引用：在编译时处理，如果方法不存在或者方法签名与监听器中的方法签名不匹配，则触发编译时错误；在数据绑定时进行创建监听器，如果表达式求值为null，则不创建，否则创建；
- 监听器绑定：在触发事件时，对表达式进行求值；只要返回类型匹配就可以；

#### 1. `@{click}`

```
//xml:
<Button
     android:layout_width="match_parent"
     android:layout_height="48dp"
     android:onClick="@{click}"
/>

//ViewModel:
public void click(View view){

}
```

#### 2. 不带参数：`@{() -> viewModel.click()}`

```
//xml:
<Button
     android:layout_width="match_parent"
     android:layout_height="48dp"
     android:onClick="@{() -> viewModel.click()}"
/>

//ViewModel:
public void click(){

}
```

#### 3. `@{viewModel::click}`

```
//xml:
<Button
     android:layout_width="match_parent"
     android:layout_height="48dp"
     android:onClick="@{viewModel::click}"
/>

//ViewModel:
public void click(View view){

}
```

**tip:** 如果是在其它类中设置点击方法，如 EventHandlers.java, 其实与上面一致

```
//xml:
<variable
    name="handler"
    type="com.xx.xxx.EventHandlers" />

<Button
     android:layout_width="match_parent"
     android:layout_height="48dp"
     android:onClick="@{handler::click}" />

// UI类：绑定handler,如绑定ViewModel那样
EventHandlers handler = new EventHandlers();
binding.setHandler(handler);

// EventHandlers 执行click事件
public void click(View view){
    //do
}
```

#### 4. 带参数：`@{() -> viewModel.click(obj.id)}`

```
//xml:
<variable
    name="viewModel"
    type="com.xx.xxx.ViewModel" />

<variable
    name="obj"
    type="com.xx.xxx.User" />
<Button
     android:layout_width="match_parent"
     android:layout_height="48dp"
     android:onClick="@{() -> viewModel.click(obj.id)}"
/>

//ViewModel:
public void click(long id){
     //do   
}
```

#### 5. `ObservableField<OnClickListener>`

```
//xml:
  <variable
       name="iconView"
       type="com.xxxxx.IconView" />

  <RelativeLayout
        android:id="@+id/rl_icon_view"
        android:layout_width="match_parent"
        android:layout_height="55dp"
        android:onClick="@{iconView.clickListener}"
       >    

//IconView:
  public final ObservableField<OnClickListener> clickListener=new ObservableField<>();

//xml所在的Activity类：
  binding.iconView.clickListener.set(new View.OnClickListener() {
         @Override
         public void onClick(View v) {
              Toast.makeText(getApplicationContext(),"iconView",Toast.LENGTH_SHORT).show();
         }
  });
```

#### 6. 带参数

```
 android:onClick="@{()->loadingModel.chooseLang(2)}"
 
  public void chooseLang(int language) {
        LogUtil.i(TAG, "language=" + language);

  }
```

#### 7. 带view的参数

```
//xml:
<variable
       name="listener"
       type="test.carrie.todomvvmtest.ui.OnTaskItemListener"/> 

android:onClick="@{(view)->listener.onCheckBoxClick(obj,view)}"


//adapter:
 public void onCheckBoxClick(ToDo entity, View v) {

 }
```

### include嵌套

- 当XML中通过include嵌套了另一个XML时，可以通过`bind:` + 被嵌套XML中声明的变量名将XML中的数据传给被嵌套的XML；

**buttons.xml:**

```java
<layout xmlns:andr...>
  <data>
    <variable name="foo" type="int"/>
  </data>
  <Button
    android:id="@+id/button"
    ...." />
```

**main.xml:**

```java
<layout xmlns:andr...
...
   <include layout="@layout/buttons"
            android:id="@+id/buttons"
            app:foo="@{1}"/>
....
```

Then you can access buttons indirectly through the buttons field:

```java
MainBinding binding = MainBinding.inflate(getLayoutInflater());
binding.buttons.button
```

### 双向绑定

上面的四类注解是单向绑定需要的，双向绑定是基于单向绑定的；

InverseBindingAdapter

```csharp
@InverseBindingAdapter("attribute" = attrName, event = attrName + "AttrChange")
public static Type getAttr(View view, Type old);
```

- 注解在方法上，用于从View组件获取当前状态；event表示监听事件，需要另外的BindingAdapter(event)方法对View进行监听绑定；

InverseBindingMethods/InverseBindingMethod

```csharp
InverseBindingMethods({InverseBindingMethod(
    type =android.widget.TextView.class, 
    attribute = "android:text", 
    event = "android:textAttrChanged",
    method = "getText")})
```

- 和InverseBindingAdapter类似，不需要定义获取View状态的getter方法，同样需要定义BindingAdapter(event)对View进行监听绑定；

InverseMethod

```csharp
InverseMethod("convertIntToString")
public static int convertStringToInt(String value);
```

- 类似于单向绑定中的BindingConversion，逆向转换，但是参数类型，返回类型要和对应的BindingConversion正好相反；

### DataBindingComponent

- 以上注解定义的都是针对全局的方法，如果只是某个XML中需要用到特殊的方法进行数据绑定，那么可以通过一个类实现了DataBindingComponent接口（空接口），并且在获取binding对象时，传入该类的对象即可；
- 个人不建议采用这种方式，如果某个XML有特殊的数据绑定逻辑，可以通过在可观察数据源上挂监听更新UI（不用DataBinding自动更新部分）；

## 常见问题

### 1.[Set drawable resource ID in android:src for ImageView using data binding in Android](https://stackoverflow.com/questions/35809290/set-drawable-resource-id-in-androidsrc-for-imageview-using-data-binding-in-andr)

```java
public class DataBindingAdapters {

    @BindingAdapter("android:src")
    public static void setImageUri(ImageView view, String imageUri) {
        if (imageUri == null) {
            view.setImageURI(null);
        } else {
            view.setImageURI(Uri.parse(imageUri));
        }
    }

    @BindingAdapter("android:src")
    public static void setImageUri(ImageView view, Uri imageUri) {
        view.setImageURI(imageUri);
    }

    @BindingAdapter("android:src")
    public static void setImageDrawable(ImageView view, Drawable drawable) {
        view.setImageDrawable(drawable);
    }

    @BindingAdapter("android:src")
    public static void setImageResource(ImageView imageView, int resource){
        imageView.setImageResource(resource);
    }
}
```

## 参考

* [DataBinding--布局和绑定表达式](https://developer.android.com/topic/libraries/data-binding/expressions)
* [DataBinding--使用可观察的数据对象](https://developer.android.com/topic/libraries/data-binding/observability)
* [DataBinding--生成的绑定类](https://developer.android.com/topic/libraries/data-binding/generated-binding)
* [DataBinding--绑定适配器](https://developer.android.com/topic/libraries/data-binding/binding-adapters)
* [DataBinding--将布局视图绑定到架构组件](https://developer.android.com/topic/libraries/data-binding/architecture)

