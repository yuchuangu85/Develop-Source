<h1 align="center">Android窗口管理框架：Android布局解析者LayoutInflater</h1>

**文章目录**

- 一 获取XmlResourceParser
- 二 解析View树
- 三 解析View

>Instantiates a layout XML file into its corresponding {@link android.view.View}objects. 

LayoutInflater可以把xml布局文件里内容加载成一个View，LayoutInflater可以说是Android里的无名英雄，你经常用的到它，却体会不到它的好。因为隔壁的iOS兄弟是没有
这种东西的，他们只能用代码来写布局，需要应用跑起来才能看到效果。相比之下Android的开发者就幸福的多，但是大家有没有相关xml是如何转换成一个View的，今天我们就来分析
这个问题。

LayoutInflater也是通过Context获取，它也是系统服务的一种，被注册在ContextImpl的map里，然后通过LAYOUT_INFLATER_SERVICE来获取。

```java
layoutInflater = (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
```
LayoutInflater是一个抽象类，它的实现类是PhoneLayoutInflater。LayoutInflater会采用深度优先遍历自顶向下遍历View树，根据View的全路径名利用反射获取构造器
从而构建View的实例。整个逻辑还是很清晰的，我们来具体看一看。

<img src="../../../art/app/ui/LayoutInflater_sequence.png"/>

我们先来看看总的调度方法inflate()，这个也是我们最常用的

```java
public View inflate(@LayoutRes int resource, @Nullable ViewGroup root, boolean attachToRoot)
```
这个方法有三个参数：

int resource：布局ID，也就是要解析的xml布局文件，boolean attachToRoot表示是否要添加到父布局root中去。这里面还有个关键的参数root。它用来表示根布局，这个就很常见的，我们在用
这个方法的时候，有时候给root赋值了，有时候直接给了null（给null的时候IDE会有警告提示），这个root到底有什么作用呢？🤔

它主要有两个方面的作用：

- 当attachToRoot == true且root ！= null时，新解析出来的View会被add到root中去，然后将root作为结果返回。
- 当attachToRoot == false且root ！= null时，新解析的View会直接作为结果返回，而且root会为新解析的View生成LayoutParams并设置到该View中去。
- 当attachToRoot == false且root == null时，新解析的View会直接作为结果返回。

注意第二条和第三条是由区别的，你可以去写个例子试一下，当root为null时，新解析出来的View没有LayoutParams参数，这时候你设置的layout_width和layout_height是不生效的。

说到这里，有人可能有疑问了，Activity里的布局应该也是LayoutInflater加载的，我也没做什么处理，但是我设置的layout_width和layout_heigh参数都是可以生效的，这是为什么？🤔

>这是因为Activity内部做了处理，我们知道Activity的setContentView()方法，实际上调用的PhoneWindow的setContentView()方法。它调用的时候将Activity的顶级DecorView（FrameLayout）
作为root传了进去，mLayoutInflater.inflate(layoutResID, mContentParent)实际调用的是inflate(resource, root, root != null)，所以在调用Activity的setContentView()方法时
可以将解析出的View添加到顶级DecorView中，我们设置的layout_width和layout_height参数也可以生效。

具体代码如下：

```java
@Override
public void setContentView(int layoutResID) {
    if (mContentParent == null) {
        installDecor();
    } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
        mContentParent.removeAllViews();
    }

    if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
        final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID,
                getContext());
        transitionTo(newScene);
    } else {
        
        mLayoutInflater.inflate(layoutResID, mContentParent);
    }
    mContentParent.requestApplyInsets();
    final Callback cb = getCallback();
    if (cb != null && !isDestroyed()) {
        cb.onContentChanged();
    }
    mContentParentExplicitlySet = true;
}
```

了解了inflate()方法各个参数的含义，我们正式来分析它的实现。

```java

public abstract class LayoutInflater {

    public View inflate(@LayoutRes int resource, @Nullable ViewGroup root, boolean attachToRoot) {
        final Resources res = getContext().getResources();
        if (DEBUG) {
            Log.d(TAG, "INFLATING from resource: \"" + res.getResourceName(resource) + "\" ("
                    + Integer.toHexString(resource) + ")");
        }
        
        //获取xml资源解析器XmlResourceParser
        final XmlResourceParser parser = res.getLayout(resource);
        try {
            return inflate(parser, root, attachToRoot);//解析View
        } finally {
            parser.close();
        }
    }
}
```

可以发现在该方法里，主要完成了两件事情：

1. 获取xml资源解析器XmlResourceParser。
2. 解析View

我们先来看看XmlResourceParser是如何获取的。

从上面的序列图可以看出，调用了Resources的getLayout(resource)去获取对应的XmlResourceParser。getLayout(resource)又去调用了Resources的loadXmlResourceParser()
方法来完成XmlResourceParser的加载，如下所示：

```java
public class Resources {
    
     XmlResourceParser loadXmlResourceParser(@AnyRes int id, @NonNull String type)
             throws NotFoundException {
         final TypedValue value = obtainTempTypedValue();
         try {
             final ResourcesImpl impl = mResourcesImpl;
             //1. 获取xml布局资源，并保存在TypedValue中。
             impl.getValue(id, value, true);
             if (value.type == TypedValue.TYPE_STRING) {
                 //2. 加载对应的loadXmlResourceParser解析器。
                 return impl.loadXmlResourceParser(value.string.toString(), id,
                         value.assetCookie, type);
             }
             throw new NotFoundException("Resource ID #0x" + Integer.toHexString(id)
                     + " type #0x" + Integer.toHexString(value.type) + " is not valid");
         } finally {
             releaseTempTypedValue(value);
         }
     }   
}
```

可以发现这个方法又被分成了两步：

1. 获取xml布局资源，并保存在TypedValue中。
2. 加载对应的loadXmlResourceParser解析器。

从上面的序列图可以看出，资源的获取涉及到resources.arsc的解析过程，这个我们已经在**Resources的创建流程**简单聊过，这里就不再赘述。通过
getValue()方法获取到xml资源以后，就会调用ResourcesImpl的loadXmlResourceParser()方法对该布局资源进行解析，以便得到一个UI布局视图。

我们来看看它的实现。

## 一 获取XmlResourceParser

```java
public class ResourcesImpl {
    
    XmlResourceParser loadXmlResourceParser(@NonNull String file, @AnyRes int id, int assetCookie,
               @NonNull String type)
               throws NotFoundException {
           if (id != 0) {
               try {
                   synchronized (mCachedXmlBlocks) {
                       //... 从缓存中查找xml资源
   
                       // Not in the cache, create a new block and put it at
                       // the next slot in the cache.
                       final XmlBlock block = mAssets.openXmlBlockAsset(assetCookie, file);
                       if (block != null) {
                           final int pos = (mLastCachedXmlBlockIndex + 1) % num;
                           mLastCachedXmlBlockIndex = pos;
                           final XmlBlock oldBlock = cachedXmlBlocks[pos];
                           if (oldBlock != null) {
                               oldBlock.close();
                           }
                           cachedXmlBlockCookies[pos] = assetCookie;
                           cachedXmlBlockFiles[pos] = file;
                           cachedXmlBlocks[pos] = block;
                           return block.newParser();
                       }
                   }
               } catch (Exception e) {
                   final NotFoundException rnf = new NotFoundException("File " + file
                           + " from xml type " + type + " resource ID #0x" + Integer.toHexString(id));
                   rnf.initCause(e);
                   throw rnf;
               }
           }
   
           throw new NotFoundException("File " + file + " from xml type " + type + " resource ID #0x"
                   + Integer.toHexString(id));
       } 
}
```

我们先来看看这个方法的四个形参：

- String file：xml文件的路径
- int id：xml文件的资源ID
- int assetCookie：xml文件的资源缓存
- String type：资源类型

ResourcesImpl会缓存最近解析的4个xml资源，如果不在缓存里则调用AssetManger的openXmlBlockAsset()方法创建一个XmlBlock。XmlBlock是已编译的xml文件的一个包装类。

AssetManger的openXmlBlockAsset()方法的实现如下所示：

```java
public final class AssetManager implements AutoCloseable {
   /*package*/ final XmlBlock openXmlBlockAsset(int cookie, String fileName)
       throws IOException {
       synchronized (this) {
           //...
           long xmlBlock = openXmlAssetNative(cookie, fileName);
           if (xmlBlock != 0) {
               XmlBlock res = new XmlBlock(this, xmlBlock);
               incRefsLocked(res.hashCode());
               return res;
           }
       }
       //...
   } 
}
```

可以看出该方法会调用native方法openXmlAssetNative()去代开fileName指定的xml文件，成功打开该文件后，会得到C++层的ResXMLTree对象的地址xmlBlock，然后将xmlBlock封装进
XmlBlock中返回给调用者。ResXMLTreed对象会存放打开后xml资源的内容。

上述序列图里的AssetManger.cpp的方法的具体实现也就是一个打开资源文件的过程，资源文件一般存放在APK中，APK是一个zip包，所以最终会调用openAssetFromZipLocked()方法打开xml文件。

XmlBlock封装完成后，会调用XmlBlock对象的newParser()方法去构建一个XmlResourceParser对象，实现如下所示：

```java
final class XmlBlock {
    public XmlResourceParser newParser() {
        synchronized (this) {
            //mNative指向的是C++层的ResXMLTree对象的地址
            if (mNative != 0) {
                return new Parser(nativeCreateParseState(mNative), this);
            }
            return null;
        }
    }
    
    private static final native long nativeCreateParseState(long obj);
}
```

mNative指向的是C++层的ResXMLTree对象的地址，native方法nativeCreateParseState()根据这个地址找到ResXMLTree对象，利用ResXMLTree对象对象构建一个ResXMLParser对象，并将ResXMLParser对象
的地址封装进Java层的Parser对象中，构建一个Parser对象。所以他们的对应关系如下所示：

- XmlBlock <--> ResXMLTree
- Parser <--> ResXMLParser

就是建立了Java层与C++层的对应关系，实际的实现还是由C++层完成。

等获取了XmlResourceParser对象以后就可以调用inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot) 方法进行View的解析了，在解析View时
，会先去调用rInflate()方法解析View树，然后再调用createViewFromTag()方法创建具体的View，我们来详细的看一看。

## 二 解析View树

1. 解析merge标签，rInflate()方法会将merge下面的所有子View直接添加到根容器中，这里我们也理解了为什么merge标签可以达到简化布局的效果。
2. 不是merge标签那么直接调用createViewFromTag()方法解析成布局中的视图，这里的参数name就是要解析视图的类型，例如：ImageView。
3. 调用generateLayoutParams()f方法生成布局参数，如果attachToRoot为false，即不添加到根容器里，为View设置布局参数。
4. 调用rInflateChildren()方法解析当前View下面的所有子View。
5. 如果根容器不为空，且attachToRoot为true，则将解析出来的View添加到根容器中，如果根布局为空或者attachToRoot为false，那么解析出来的额View就是返回结果。返回解析出来的结果。

接下来，我们分别看下View树解析以及View的解析。

```java
public abstract class LayoutInflater {
    
    public View inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot) {
           synchronized (mConstructorArgs) {
               Trace.traceBegin(Trace.TRACE_TAG_VIEW, "inflate");
   
               final Context inflaterContext = mContext;
               final AttributeSet attrs = Xml.asAttributeSet(parser);
               
               //Context对象
               Context lastContext = (Context) mConstructorArgs[0];
               mConstructorArgs[0] = inflaterContext;
               
               //存储根视图
               View result = root;
   
               try {
                   // 获取根元素
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
                   
                   if (DEBUG) {
                       System.out.println("**************************");
                       System.out.println("Creating root view: "
                               + name);
                       System.out.println("**************************");
                   }
   
                   //1. 解析merge标签，rInflate()方法会将merge下面的所有子View直接添加到根容器中，这里
                   //我们也理解了为什么merge标签可以达到简化布局的效果。
                   if (TAG_MERGE.equals(name)) {
                       if (root == null || !attachToRoot) {
                           throw new InflateException("<merge /> can be used only with a valid "
                                   + "ViewGroup root and attachToRoot=true");
                       }
   
                       rInflate(parser, root, inflaterContext, attrs, false);
                   } else {
                       //2. 不是merge标签那么直接调用createViewFromTag()方法解析成布局中的视图，这里的参数name就是要解析视图的类型，例如：ImageView
                       final View temp = createViewFromTag(root, name, inflaterContext, attrs);
   
                       ViewGroup.LayoutParams params = null;
   
                       if (root != null) {
                           if (DEBUG) {
                               System.out.println("Creating params from root: " +
                                       root);
                           }
                           //3. 调用generateLayoutParams()f方法生成布局参数，如果attachToRoot为false，即不添加到根容器里，为View设置布局参数
                           params = root.generateLayoutParams(attrs);
                           if (!attachToRoot) {
                               // Set the layout params for temp if we are not
                               // attaching. (If we are, we use addView, below)
                               temp.setLayoutParams(params);
                           }
                       }
   
                       if (DEBUG) {
                           System.out.println("-----> start inflating children");
                       }
   
                       //4. 调用rInflateChildren()方法解析当前View下面的所有子View
                       rInflateChildren(parser, temp, attrs, true);
   
                       if (DEBUG) {
                           System.out.println("-----> done inflating children");
                       }
   
                       //如果根容器不为空，且attachToRoot为true，则将解析出来的View添加到根容器中
                       if (root != null && attachToRoot) {
                           root.addView(temp, params);
                       }
   
                       //如果根布局为空或者attachToRoot为false，那么解析出来的额View就是返回结果
                       if (root == null || !attachToRoot) {
                           result = temp;
                       }
                   }
   
               } catch (XmlPullParserException e) {
                   final InflateException ie = new InflateException(e.getMessage(), e);
                   ie.setStackTrace(EMPTY_STACK_TRACE);
                   throw ie;
               } catch (Exception e) {
                   final InflateException ie = new InflateException(parser.getPositionDescription()
                           + ": " + e.getMessage(), e);
                   ie.setStackTrace(EMPTY_STACK_TRACE);
                   throw ie;
               } finally {
                   // Don't retain static reference on context.
                   mConstructorArgs[0] = lastContext;
                   mConstructorArgs[1] = null;
   
                   Trace.traceEnd(Trace.TRACE_TAG_VIEW);
               }
   
               return result;
           }
     }
}
```

上面我们已经提到View树的解析是有rInflate()方法来完成的，我们接着来看看View树是如何被解析的。

```java
public abstract class LayoutInflater {
    
    void rInflate(XmlPullParser parser, View parent, Context context,
            AttributeSet attrs, boolean finishInflate) throws XmlPullParserException, IOException {

        //1. 获取树的深度，执行深度优先遍历
        final int depth = parser.getDepth();
        int type;

        //2. 逐个进行元素解析
        while (((type = parser.next()) != XmlPullParser.END_TAG ||
                parser.getDepth() > depth) && type != XmlPullParser.END_DOCUMENT) {

            if (type != XmlPullParser.START_TAG) {
                continue;
            }

            final String name = parser.getName();
            
            if (TAG_REQUEST_FOCUS.equals(name)) {
                //3. 解析添加ad:focusable="true"的元素，并获取View焦点。
                parseRequestFocus(parser, parent);
            } else if (TAG_TAG.equals(name)) {
                //4. 解析View的tag。
                parseViewTag(parser, parent, attrs);
            } else if (TAG_INCLUDE.equals(name)) {
                //5. 解析include标签，注意include标签不能作为根元素。
                if (parser.getDepth() == 0) {
                    throw new InflateException("<include /> cannot be the root element");
                }
                parseInclude(parser, context, parent, attrs);
            } else if (TAG_MERGE.equals(name)) {
                //merge标签必须为根元素
                throw new InflateException("<merge /> must be the root element");
            } else {
                //6. 根据元素名进行解析，生成View。
                final View view = createViewFromTag(parent, name, context, attrs);
                final ViewGroup viewGroup = (ViewGroup) parent;
                final ViewGroup.LayoutParams params = viewGroup.generateLayoutParams(attrs);
                //7. 递归调用解析该View里的所有子View，也是深度优先遍历，rInflateChildren内部调用的也是rInflate()方
                //法，只是传入了新的parent View
                rInflateChildren(parser, view, attrs, true);
                //8. 将解析出来的View添加到它的父View中。
                viewGroup.addView(view, params);
            }
        }

        if (finishInflate) {
            //9. 回调根容器的onFinishInflate()方法，这个方法我们应该很熟悉。
            parent.onFinishInflate();
        }
    }
    
    //rInflateChildren内部调用的也是rInflate()方法，只是传入了新的parent View
    final void rInflateChildren(XmlPullParser parser, View parent, AttributeSet attrs,
            boolean finishInflate) throws XmlPullParserException, IOException {
        rInflate(parser, parent, parent.getContext(), attrs, finishInflate);
    }

}
```

上述方法描述了整个View树的解析流程，我们来概括一下：

1. 获取树的深度，执行深度优先遍历.
2. 逐个进行元素解析。
3. 解析添加ad:focusable="true"的元素，并获取View焦点。
4. 解析View的tag。
5. 解析include标签，注意include标签不能作为根元素，而merge必须作为根元素。
6. 根据元素名进行解析，生成View。
7. 递归调用解析该View里的所有子View，也是深度优先遍历，rInflateChildren内部调用的也是rInflate()方法，只是传入了新的parent View。
8. 将解析出来的View添加到它的父View中。
9. 回调根容器的onFinishInflate()方法，这个方法我们应该很熟悉。

你可以看到，负责解析单个View的正是createViewFromTag()方法，我们再来分析下这个方法。

## 三 解析View

```java
public abstract class LayoutInflater {

        View createViewFromTag(View parent, String name, Context context, AttributeSet attrs,
                boolean ignoreThemeAttr) {
            
            //1. 解析view标签。注意是小写view，这个不太常用，下面会说。
            if (name.equals("view")) {
                name = attrs.getAttributeValue(null, "class");
            }
    
            //2. 如果标签与主题相关，则需要将context与themeResId包裹成ContextThemeWrapper。
            if (!ignoreThemeAttr) {
                final TypedArray ta = context.obtainStyledAttributes(attrs, ATTRS_THEME);
                final int themeResId = ta.getResourceId(0, 0);
                if (themeResId != 0) {
                    context = new ContextThemeWrapper(context, themeResId);
                }
                ta.recycle();
            }
    
            //3. BlinkLayout是一种会闪烁的布局，被包裹的内容会一直闪烁，像QQ消息那样。
            if (name.equals(TAG_1995)) {
                // Let's party like it's 1995!
                return new BlinkLayout(context, attrs);
            }
    
            try {
                View view;
                
                //4. 用户可以设置LayoutInflater的Factory来进行View的解析，但是默认情况下
                //这些Factory都是为空的。
                if (mFactory2 != null) {
                    view = mFactory2.onCreateView(parent, name, context, attrs);
                } else if (mFactory != null) {
                    view = mFactory.onCreateView(name, context, attrs);
                } else {
                    view = null;
                }
                if (view == null && mPrivateFactory != null) {
                    view = mPrivateFactory.onCreateView(parent, name, context, attrs);
                }
    
                //5. 默认情况下没有Factory，而是通过onCreateView()方法对内置View进行解析，createView()
                //方法进行自定义View的解析。
                if (view == null) {
                    final Object lastContext = mConstructorArgs[0];
                    mConstructorArgs[0] = context;
                    try {
                        //这里有个小技巧，因为我们在使用自定义View的时候是需要在xml指定全路径的，例如：
                        //com.guoxiaoxing.CustomView，那么这里就有个.了，可以利用这一点判定是内置View
                        //还是自定义View，Google的工程师很机智的。😎
                        if (-1 == name.indexOf('.')) {
                            //内置View解析
                            view = onCreateView(parent, name, attrs);
                        } else {
                            //自定义View解析
                            view = createView(name, null, attrs);
                        }
                    } finally {
                        mConstructorArgs[0] = lastContext;
                    }
                }
    
                return view;
            } catch (InflateException e) {
                throw e;
    
            } catch (ClassNotFoundException e) {
                final InflateException ie = new InflateException(attrs.getPositionDescription()
                        + ": Error inflating class " + name, e);
                ie.setStackTrace(EMPTY_STACK_TRACE);
                throw ie;
    
            } catch (Exception e) {
                final InflateException ie = new InflateException(attrs.getPositionDescription()
                        + ": Error inflating class " + name, e);
                ie.setStackTrace(EMPTY_STACK_TRACE);
                throw ie;
            }
        }
}
```

单个View的解析流程也很简单，我们来梳理一下：

1. 解析View标签。
2. 如果标签与主题相关，则需要将context与themeResId包裹成ContextThemeWrapper。
3. BlinkLayout是一种会闪烁的布局，被包裹的内容会一直闪烁，像QQ消息那样。
4. 用户可以设置LayoutInflater的Factory来进行View的解析，但是默认情况下这些Factory都是为空的。
5. 默认情况下没有Factory，而是通过onCreateView()方法对内置View进行解析，createView()方法进行自定义View的解析。这里有个小技巧，因为
我们在使用自定义View的时候是需要在xml指定全路径的，例如：com.guoxiaoxing.CustomView，那么这里就有个.了，可以利用这一点判定是内置View
还是自定义View，Google的工程师很机智的。😎

**关于view标签**

```xml
<view
    class="RelativeLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"/>
```
在使用时，相当于所有控件标签的父类一样，可以设置class属性，这个属性会决定view这个节点会是什么控件。

**关于BlinkLayout**

这个也是个冷门的控件，本质上是一个FrameLayout，被它包裹的控件会像电脑版的QQ小企鹅那样一直闪烁。

```xml
<blink
    android:layout_width="wrap_content"
    android:layout_height="wrap_content">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="这个控件会一直闪烁"/>

</blink>
```

**关于onCreateView()与createView()**

这两个方法在本质上都是一样的，只是onCreateView()会给内置的View前面加一个前缀，例如：android.widget，方便开发者在写内置View的时候，不用谢全路径名。
前面我们也提到了LayoutInflater是一个抽象类，我们实际使用的PhoneLayoutInflater，这个类的实现很简单，它重写了LayoutInflater的onCreatView()方法，该
方法就是做了一个给内置View加前缀的事情。

```java
public class PhoneLayoutInflater extends LayoutInflater {
    private static final String[] sClassPrefixList = {
        "android.widget.",
        "android.webkit.",
        "android.app."
    };

    @Override protected View onCreateView(String name, AttributeSet attrs) throws ClassNotFoundException {
        
        //循环遍历三种前缀，尝试创建View
        for (String prefix : sClassPrefixList) {
            try {
                View view = createView(name, prefix, attrs);
                if (view != null) {
                    return view;
                }
            } catch (ClassNotFoundException e) {
                // In this case we want to let the base class take a crack
                // at it.
            }
        }
        return super.onCreateView(name, attrs);
    }
    public LayoutInflater cloneInContext(Context newContext) {
        return new PhoneLayoutInflater(this, newContext);
    }
}
```
这样一来，真正的View构建还是在createView()方法里完成的，createView()主要根据完整的类的路径名利用反射机制构建View对象，我们具体来
看看createView()方法的实现。

```java
public abstract class LayoutInflater {
    
    public final View createView(String name, String prefix, AttributeSet attrs)
                throws ClassNotFoundException, InflateException {
        
            //1. 从缓存中读取构造函数。
            Constructor<? extends View> constructor = sConstructorMap.get(name);
            if (constructor != null && !verifyClassLoader(constructor)) {
                constructor = null;
                sConstructorMap.remove(name);
            }
            Class<? extends View> clazz = null;
    
            try {
                Trace.traceBegin(Trace.TRACE_TAG_VIEW, name);
    
                if (constructor == null) {
                    // Class not found in the cache, see if it's real, and try to add it
                    
                    //2. 没有在缓存中查找到构造函数，则构造完整的路径名，并加装该类。
                    clazz = mContext.getClassLoader().loadClass(
                            prefix != null ? (prefix + name) : name).asSubclass(View.class);
                    
                    if (mFilter != null && clazz != null) {
                        boolean allowed = mFilter.onLoadClass(clazz);
                        if (!allowed) {
                            failNotAllowed(name, prefix, attrs);
                        }
                    }
                    //3. 从Class对象中获取构造函数，并在sConstructorMap做下缓存，方便下次使用。
                    constructor = clazz.getConstructor(mConstructorSignature);
                    constructor.setAccessible(true);
                    sConstructorMap.put(name, constructor);
                } else {
                    //4. 如果sConstructorMap中有当前View构造函数的缓存，则直接使用。
                    if (mFilter != null) {
                        // Have we seen this name before?
                        Boolean allowedState = mFilterMap.get(name);
                        if (allowedState == null) {
                            // New class -- remember whether it is allowed
                            clazz = mContext.getClassLoader().loadClass(
                                    prefix != null ? (prefix + name) : name).asSubclass(View.class);
                            
                            boolean allowed = clazz != null && mFilter.onLoadClass(clazz);
                            mFilterMap.put(name, allowed);
                            if (!allowed) {
                                failNotAllowed(name, prefix, attrs);
                            }
                        } else if (allowedState.equals(Boolean.FALSE)) {
                            failNotAllowed(name, prefix, attrs);
                        }
                    }
                }
    
                Object[] args = mConstructorArgs;
                args[1] = attrs;
    
                //5. 利用构造函数，构建View对象。
                final View view = constructor.newInstance(args);
                if (view instanceof ViewStub) {
                    // Use the same context when inflating ViewStub later.
                    final ViewStub viewStub = (ViewStub) view;
                    viewStub.setLayoutInflater(cloneInContext((Context) args[0]));
                }
                return view;
    
            } catch (NoSuchMethodException e) {
                final InflateException ie = new InflateException(attrs.getPositionDescription()
                        + ": Error inflating class " + (prefix != null ? (prefix + name) : name), e);
                ie.setStackTrace(EMPTY_STACK_TRACE);
                throw ie;
    
            } catch (ClassCastException e) {
                // If loaded class is not a View subclass
                final InflateException ie = new InflateException(attrs.getPositionDescription()
                        + ": Class is not a View " + (prefix != null ? (prefix + name) : name), e);
                ie.setStackTrace(EMPTY_STACK_TRACE);
                throw ie;
            } catch (ClassNotFoundException e) {
                // If loadClass fails, we should propagate the exception.
                throw e;
            } catch (Exception e) {
                final InflateException ie = new InflateException(
                        attrs.getPositionDescription() + ": Error inflating class "
                                + (clazz == null ? "<unknown>" : clazz.getName()), e);
                ie.setStackTrace(EMPTY_STACK_TRACE);
                throw ie;
            } finally {
                Trace.traceEnd(Trace.TRACE_TAG_VIEW);
            }
        }   
}
```

好了，到这篇文章为止，我们对整个Android显示框架的原理分析就算是告一段落了，在这些文章里我们侧重的是Client端的分析，WindowManagerService、SurfaceFlinger这些Server端的
并没有过多的涉及，因为对大部分开发者而言，扎实的掌握Client端的原理就足够了。等到你完全掌握了Client端的原理或者是需要进行Android Framework层的开发，可以进一步去深入Server
端的原理。

关于Android显示框架主要包括五篇文章：

- [01Android显示框架：Android显示框架概述](https://github.com/guoxiaoxing/android-open-source-project-analysis/blob/master/doc/Android系统应用框架篇/Android显示框架/01Android显示框架：Android显示框架概述.md)
- [02Android显示框架：Android应用视图的载体View](https://github.com/guoxiaoxing/android-open-source-project-analysis/blob/master/doc/Android系统应用框架篇/Android显示框架/02Android显示框架：Android应用视图载体View.md)
- [03Android显示框架：Android应用视图的管理者Window](https://github.com/guoxiaoxing/android-open-source-project-analysis/blob/master/doc/Android系统应用框架篇/Android显示框架/03Android显示框架：Android应用视图管理者Window.md)
- [04Android显示框架：Android应用窗口管理者WindowManager](https://github.com/guoxiaoxing/android-open-source-project-analysis/blob/master/doc/Android系统应用框架篇/Android显示框架/04Android显示框架：Android应用窗口管理者WindowManager.md)
- [05Android显示框架：Android布局解析者LayoutInflater](https://github.com/guoxiaoxing/android-open-source-project-analysis/blob/master/doc/Android系统应用框架篇/Android显示框架/05Android显示框架：Android布局解析者LayoutInflater.md)

后续我们会接着进行

- Android组件框架
- Android动画框架
- Android通信框架
- Android多媒体框架

等Android子系统的分析，后续的内容可以关注[Android open source project analysis](https://github.com/guoxiaoxing/android-open-source-project-analysis)项目。