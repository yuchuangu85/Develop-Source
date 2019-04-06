# Merge使用
首先我们看官方的说明:

The <merge /> tag helps eliminate redundant view groups in your view hierarchy when including one layout within another. For example, if your main layout is a vertical LinearLayout in which two consecutive views can be re-used in multiple layouts, then the re-usable layout in which you place the two views requires its own root view. However, using another LinearLayout as the root for the re-usable layout would result in a vertical LinearLayout inside a vertical LinearLayout. The nested LinearLayout serves no real purpose other than to slow down your UI performance.

其实就是减少在include布局文件时的层级。\<merge\>标签是这几个标签中最让我费解的，大家可能想不到，<merge>标签竟然会是一个Activity，里面有一个LinearLayout对象。

使用merge来组织子元素可以减少布局的层级。例如我们在复用一个含有多个子控件的布局时，肯定需要一个ViewGroup来管理，例如这样 : 

```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    android:layout_width="fill_parent"  
    android:layout_height="fill_parent">  
  
    <ImageView    
        android:layout_width="fill_parent"   
        android:layout_height="fill_parent"   
        android:scaleType="center"  
        android:src="@drawable/golden_gate" />  
      
    <TextView  
        android:layout_width="wrap_content"   
        android:layout_height="wrap_content"   
        android:layout_marginBottom="20dip"  
        android:layout_gravity="center_horizontal|bottom"  
        android:padding="12dip"  
        android:background="#AA000000"  
        android:textColor="#ffffffff"  
        android:text="Golden Gate" />  
  
</FrameLayout>  
```

将该布局通过include引入时就会多引入了一个FrameLayout层级，此时结构如下 : 

使用merge标签就会消除上图中蓝色的FrameLayout层级。示例如下 : 

```xml
<merge xmlns:android="http://schemas.android.com/apk/res/android">  
  
    <ImageView    
        android:layout_width="fill_parent"   
        android:layout_height="fill_parent"   
        android:scaleType="center"  
        android:src="@drawable/golden_gate" />  
      
    <TextView  
        android:layout_width="wrap_content"   
        android:layout_height="wrap_content"   
        android:layout_marginBottom="20dip"  
        android:layout_gravity="center_horizontal|bottom"  
        android:padding="12dip"  
        android:background="#AA000000"  
        android:textColor="#ffffffff"  
        android:text="Golden Gate" />  
  
</merge>  
```

那么它是如何实现的呢，我们还是看源码吧。相关的源码也是在LayoutInflater的inflate()函数中。

```java
public View inflate(XmlPullParser parser, ViewGroup root, boolean attachToRoot) {  
       synchronized (mConstructorArgs) {  
           final AttributeSet attrs = Xml.asAttributeSet(parser);  
           Context lastContext = (Context)mConstructorArgs[0];  
           mConstructorArgs[0] = mContext;  
           View result = root;  
  
           try {  
               // Look for the root node.  
               int type;  
               while ((type = parser.next()) != XmlPullParser.START_TAG &&  
                       type != XmlPullParser.END_DOCUMENT) {  
                   // Empty  
               }  
  
               if (type != XmlPullParser.START_TAG) {  
                   throw new InflateException(parser.getPositionDescription()  
                           + ": No start tag found!");  
               }  
  
               final String name = parser.getName();  
                 
               // m如果是erge标签，那么调用rInflate进行解析  
               if (TAG_MERGE.equals(name)) {  
                   if (root == null || !attachToRoot) {  
                       throw new InflateException("<merge /> can be used only with a valid "  
                               + "ViewGroup root and attachToRoot=true");  
                   }  
                   // 解析merge标签  
                   rInflate(parser, root, attrs, false);  
               } else {  
                  // 代码省略  
               }  
  
           } catch (XmlPullParserException e) {  
               // 代码省略  
           }   
  
           return result;  
       }  
   }  
  
  
  void rInflate(XmlPullParser parser, View parent, final AttributeSet attrs,  
       boolean finishInflate) throws XmlPullParserException, IOException {  
  
   final int depth = parser.getDepth();  
   int type;  
  
   while (((type = parser.next()) != XmlPullParser.END_TAG ||  
           parser.getDepth() > depth) && type != XmlPullParser.END_DOCUMENT) {  
  
       if (type != XmlPullParser.START_TAG) {  
           continue;  
       }  
  
       final String name = parser.getName();  
         
       if (TAG_REQUEST_FOCUS.equals(name)) {  
           parseRequestFocus(parser, parent);  
       } else if (TAG_INCLUDE.equals(name)) {  
           if (parser.getDepth() == 0) {  
               throw new InflateException("<include /> cannot be the root element");  
           }  
           parseInclude(parser, parent, attrs);  
       } else if (TAG_MERGE.equals(name)) {  
           throw new InflateException("<merge /> must be the root element");  
       } else if (TAG_1995.equals(name)) {  
           final View view = new BlinkLayout(mContext, attrs);  
           final ViewGroup viewGroup = (ViewGroup) parent;  
           final ViewGroup.LayoutParams params = viewGroup.generateLayoutParams(attrs);  
           rInflate(parser, view, attrs, true);  
           viewGroup.addView(view, params);                  
       } else { // 我们的例子会进入这里  
           final View view = createViewFromTag(parent, name, attrs);  
           // 获取merge标签的parent  
           final ViewGroup viewGroup = (ViewGroup) parent;  
           // 获取布局参数  
           final ViewGroup.LayoutParams params = viewGroup.generateLayoutParams(attrs);  
           // 递归解析每个子元素  
           rInflate(parser, view, attrs, true);  
           // 将子元素直接添加到merge标签的parent view中  
           viewGroup.addView(view, params);  
       }  
   }  
  
   if (finishInflate) parent.onFinishInflate();  
}  
```

其实就是如果是merge标签，那么直接将其中的子元素添加到merge标签parent中，这样就保证了不会引入额外的层级。

**为什么跟布局如果是FrameLayout的可以用merge标签，就可以省略一个层级呢，其实是我们的Layout添加到系统的Window中的时候是添加到FrameLauyout布局中的，所以不需要再添加一层FrameLayout**
详细内容参照：[Android系统源码分析--View绘制流程之-inflate](http://codemx.cn/2018/11/20/AndroidOS013-View-inflate/)