<h1 align="center">MeasureSpec详解</h1>

>MeasureSpec介绍及使用

## 一、MeasureSpc类说明
 　SDK的介绍：MeasureSpc类封装了父View传递给子View的布局(layout)要求。每个MeasureSpc实例代表宽度或者高度
 　
它有三种模式：

①、UNSPECIFIED(未指定)，父元素部队自元素施加任何束缚，子元素可以得到任意想要的大小；

②、EXACTLY(完全)，父元素决定自元素的确切大小，子元素将被限定在给定的边界里而忽略它本身大小；

③、AT_MOST(至多)，子元素至多达到指定大小的值。

常用的三个函数：
static int getMode(int measureSpec) : 根据提供的测量值(格式)，提取模式(上述三个模式之一)
static int getSize(int measureSpec) : 根据提供的测量值(格式)，提取大小值(这个大小也就是我们通常所说的大小)
static int makeMeasureSpec(int size,int mode) : 根据提供的大小值和模式，创建一个测量值(格式)

MeasureSpc类源码分析 其为View.java类的内部类，路径：frameworksbasecorejavaandroidviewView.java

```
public class View implements ... {
    ...
    public static class MeasureSpec {
        private static final int MODE_SHIFT = 30; //移位位数为30
        //int类型占32位，向右移位30位，该属性表示掩码值，用来与size和mode进行"&"运算，获取对应值。
        private static final int MODE_MASK = 0x3 << MODE_SHIFT;
        //向右移位30位，其值为00 + (30位0)  , 即 0x0000(16进制表示)  
        public static final int UNSPECIFIED = 0 << MODE_SHIFT;  
        //向右移位30位，其值为01 + (30位0)  , 即0x1000(16进制表示)  
        public static final int EXACTLY     = 1 << MODE_SHIFT;  
        //向右移位30位，其值为02 + (30位0)  , 即0x2000(16进制表示)  
        public static final int AT_MOST     = 2 << MODE_SHIFT;  

        //创建一个整形值，其高两位代表mode类型，其余30位代表长或宽的实际值。可以是WRAP_CONTENT、MATCH_PARENT或具体大小exactly size  
        public static int makeMeasureSpec(int size, int mode) {  
            return size + mode;  
        }  
        //获取模式  ，与运算  
        public static int getMode(int measureSpec) {  
            return (measureSpec & MODE_MASK);  
        }  
        //获取长或宽的实际值 ，与运算  
        public static int getSize(int measureSpec) {  
            return (measureSpec & ~MODE_MASK);  
        }  
    }
    ...
}  
```

MeasureSpec类的处理思路是：
右移运算，使int 类型的高两位表示模式的实际值，其余30位表示长或宽的实际值----可以是WRAP_CONTENT、MATCH_PARENT或具体大小exactly size。

通过掩码MODE_MASK进行与运算 “&”，取得模式(mode)以及长或宽(value)的实际值。
MeasureSpec.makeMeasureSpec方法，实际上这个方法很简单：

```
public static int makeMeasureSpec(int size, int mode) {
    return size + mode;
}
```

其用法如下：

```
int w = View.MeasureSpec.makeMeasureSpec(0,View.MeasureSpec.UNSPECIFIED);
int h = View.MeasureSpec.makeMeasureSpec(0,View.MeasureSpec.UNSPECIFIED);
ssidtext.measure(w, h);
int width =ssidtext.getMeasuredWidth();
int height =ssidtext.getMeasuredHeight();
```

## 二、measure过程详解

UI框架开始绘制时，皆是从ViewRoot.java类开始绘制的：

ViewRoot类简要说明: 任何显示在设备中的窗口，例如：Activity、Dialog等，都包含一个ViewRoot实例，该类主要用来与远端 WindowManagerService交互以及控制(开始/销毁)绘制。
#### 1、开始UI绘制 ， 具体绘制方法则是：

```
//开始View绘制流程  
private void performTraversals(){  
    ...  
    //这两个值我们在后面讨论时，在回过头来看看是怎么赋值的。现在只需要记住其值MeasureSpec.makeMeasureSpec()构建的。  
    int childWidthMeasureSpec; //其值由MeasureSpec类构建 , makeMeasureSpec  
    int childHeightMeasureSpec;//其值由MeasureSpec类构建 , makeMeasureSpec  


    // Ask host how big it wants to be  
    host.measure(childWidthMeasureSpec, childHeightMeasureSpec);  
    ...  
}  
...
```

#### 2、调用measure()方法去做一些前期准备 measure()方法原型定义在View.java类中，final修饰符修饰，其不能被重载：

```
public class View implements ... {
    ...
    /**
    * This is called to find out how big a view should be. The parent
    * supplies constraint information in the width and height parameters.
    *
    * @param widthMeasureSpec Horizontal space requirements as imposed by the
    * parent
    * @param heightMeasureSpec Vertical space requirements as imposed by the
    * parent
    * @see #onMeasure(int, int)
    */
    public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
        //判断是否为强制布局，即带有“FORCE_LAYOUT”标记 以及 widthMeasureSpec或heightMeasureSpec发生了改变
        if ((mPrivateFlags & FORCE_LAYOUT) == FORCE_LAYOUT ||
            widthMeasureSpec != mOldWidthMeasureSpec ||
            heightMeasureSpec != mOldHeightMeasureSpec) {
             // first clears the measured dimension flag  
            //清除MEASURED_DIMENSION_SET标记   ，该标记会在onMeasure()方法后被设置  
            mPrivateFlags &= ~MEASURED_DIMENSION_SET;   
    
            // measure ourselves, this should set the measured dimension flag back  
            // 1、 测量该View本身的大小 ； 2 、 设置MEASURED_DIMENSION_SET标记，否则接写来会报异常。  
            onMeasure(widthMeasureSpec, heightMeasureSpec);  
    
            // flag not set, setMeasuredDimension() was not invoked, we raise  
            // an exception to warn the developer  
            if ((mPrivateFlags & MEASURED_DIMENSION_SET) != MEASURED_DIMENSION_SET) {  
                throw new IllegalStateException("onMeasure() did not set the"  
                        + " measured dimension by calling" + " setMeasuredDimension()");  
            }  
    
            mPrivateFlags |= LAYOUT_REQUIRED;  //下一步是layout了，添加LAYOUT_REQUIRED标记  
        }  
    
        mOldWidthMeasureSpec = widthMeasureSpec;   //保存值  
        mOldHeightMeasureSpec = heightMeasureSpec; //保存值  
    }  
    ...
    
}
```

参数widthMeasureSpec和heightMeasureSpec 由父View构建，表示父View给子View的测量要求。其值地构建如下:

measure()方法显示判断是否需要重新调用设置改View大小，即调用onMeasure()方法，然后操作两个标识符：

①、重置MEASURED_DIMENSION_SET ： onMeasure()方法中，需要添加该标识符，否则，会报异常；

②、添加LAYOUT_REQUIRED ： 表示需要进行layout操作。最后，保存当前的widthMeasureSpec和heightMeasureSpec值。

#### 3、调用onMeasure()方法去真正设置View的长宽值，其默认实现为：
 
```
//设置该View本身地大小
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
    getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}

//@param size参数一般表示设置了android:minHeight属性或者该View背景图片的大小值
public static int getDefaultSize(int size, int measureSpec) {
    int result = size;
    int specMode = MeasureSpec.getMode(measureSpec);
    int specSize = MeasureSpec.getSize(measureSpec);
    
    //根据不同的mode值，取得宽和高的实际值。  
    switch (specMode) {  
      case MeasureSpec.UNSPECIFIED:  //表示该View的大小父视图未定，设置为默认值  
          result = size;  
          break;  
      case MeasureSpec.AT_MOST:      //表示该View的大小由父视图指定了  
      case MeasureSpec.EXACTLY:  
          result = specSize;  
          break;  
    }  
    return result;  
}

//获得设置了android:minHeight属性或者该View背景图片的大小值， 最为该View的参考值
protected int getSuggestedMinimumWidth() {
    int suggestedMinWidth = mMinWidth; // android:minHeight
    if (mBGDrawable != null) { // 背景图片对应地Width。  
          final int bgMinWidth = mBGDrawable.getMinimumWidth();  
          if (suggestedMinWidth < bgMinWidth) {  
              suggestedMinWidth = bgMinWidth;  
          }  
    }  
    return suggestedMinWidth; 
}
//设置View在measure过程中宽和高
protected final void setMeasuredDimension(int measuredWidth, int measuredHeight) {
    mMeasuredWidth = measuredWidth;
    mMeasuredHeight = measuredHeight;
    mPrivateFlags |= MEASURED_DIMENSION_SET;  //设置了MEASURED_DIMENSION_SET标记  
}
```
 
主要功能就是根据该View属性(android:minWidth和背景图片大小)和父View对该子View的"测量要求"，设置该View的 mMeasuredWidth 和 mMeasuredHeight 值。

这儿只是一般的View类型地实现方法。一般来说，父View，也就是ViewGroup类型，都需要在重写onMeasure()方法，遍历所有子View，设置每个子View的大小。基本思想如下：遍历所有子View，设置每个子View的大小。伪代码表示为：

```
//某个ViewGroup类型的视图
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    //必须调用super.ononMeasure()或者直接调用setMeasuredDimension()方法设置该View大小，否则会报异常。
    super.onMeasure(widthMeasureSpec , heightMeasureSpec)
    
    //遍历每个子View
    for(int i = 0 ; i < getChildCount() ; i++){
        View child = getChildAt(i);
        //调用子View的onMeasure，设置他们的大小。childWidthMeasureSpec ， childHeightMeasureSpec ?
        child.onMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
    }
}
```

如何去设置每个子View的大小，基本思想也如同我们之前描述的思想：遍历所有子View，设置每个子View的大小。

```
//widthMeasureSpec 和 heightMeasureSpec 表示该父View的布局要求
//遍历每个子View，然后调用measureChild()方法去实现每个子View大小
protected void measureChildren(int widthMeasureSpec, int heightMeasureSpec) {
    final int size = mChildrenCount;
    final View[] children = mChildren;
    for (int i = 0; i < size; ++i) {
    final View child = children[i];
    if ((child.mViewFlags & VISIBILITY_MASK) != GONE) { // 不处于 “GONE” 状态
        measureChild(child, widthMeasureSpec, heightMeasureSpec);
    }
}
//测量每个子View高宽时，清楚了该View本身的边距大小，即android:padding属性 或android:paddingLeft等属性标记
protected void measureChild(View child, int parentWidthMeasureSpec,
int parentHeightMeasureSpec) {
    final LayoutParams lp = child.getLayoutParams(); // LayoutParams属性
    //设置子View的childWidthMeasureSpec属性，去除了该父View的边距值 mPaddingLeft + mPaddingRight
    final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
    mPaddingLeft + mPaddingRight, lp.width);
    //设置子View的childHeightMeasureSpec属性，去除了该父View的边距值 mPaddingTop + mPaddingBottom
    final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
    mPaddingTop + mPaddingBottom, lp.height);
    child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}
```

measureChildren()方法：遍历所有子View，调用measureChild()方法去设置该子View的属性值。

measureChild()  方法   ： 获取特定子View的widthMeasureSpec、heightMeasureSpec，调用measure()方法设置子View的实际宽高值。

getChildMeasureSpec()就是获取子View的widthMeasureSpec、heightMeasureSpec值。

```
// spec参数 表示该父View本身所占的widthMeasureSpec 或 heightMeasureSpec值
// padding参数 表示该父View的边距大小，见于android:padding属性 或android:paddingLeft等属性标记
// childDimension参数 表示该子View内部LayoutParams属性的值，可以是wrap_content、match_parent、一个精确指(an exactly size),
// 例如：由android:width指定等。
public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
    int specMode = MeasureSpec.getMode(spec); //获得父View的mode
    int specSize = MeasureSpec.getSize(spec); //获得父View的实际值
    int size = Math.max(0, specSize - padding); //父View为子View设定的大小，减去边距值，  
    
    int resultSize = 0;    //子View对应地 size 实际值 ，由下面的逻辑条件赋值  
    int resultMode = 0;    //子View对应地 mode 值 ， 由下面的逻辑条件赋值  
    
    switch (specMode) {  
        // Parent has imposed an exact size on us  
        //1、父View是EXACTLY的 ！  
        case MeasureSpec.EXACTLY:   
            //1.1、子View的width或height是个精确值 (an exactly size)  
            if (childDimension >= 0) {            
                resultSize = childDimension;         //size为精确值  
                resultMode = MeasureSpec.EXACTLY;    //mode为 EXACTLY 。  
            }   
            //1.2、子View的width或height为 MATCH_PARENT/FILL_PARENT   
            else if (childDimension == LayoutParams.MATCH_PARENT) {  
                // Child wants to be our size. So be it.  
                resultSize = size;                   //size为父视图大小  
                resultMode = MeasureSpec.EXACTLY;    //mode为 EXACTLY 。  
            }   
            //1.3、子View的width或height为 WRAP_CONTENT  
            else if (childDimension == LayoutParams.WRAP_CONTENT) {  
                // Child wants to determine its own size. It can't be  
                // bigger than us.  
                resultSize = size;                   //size为父视图大小  
                resultMode = MeasureSpec.AT_MOST;    //mode为AT_MOST 。  
            }  
            break;  
        
        // Parent has imposed a maximum size on us  
        //2、父View是AT_MOST的 ！      
        case MeasureSpec.AT_MOST:  
            //2.1、子View的width或height是个精确值 (an exactly size)  
            if (childDimension >= 0) {  
                // Child wants a specific size... so be it  
                resultSize = childDimension;        //size为精确值  
                resultMode = MeasureSpec.EXACTLY;   //mode为 EXACTLY 。  
            }  
            //2.2、子View的width或height为 MATCH_PARENT/FILL_PARENT  
            else if (childDimension == LayoutParams.MATCH_PARENT) {  
                // Child wants to be our size, but our size is not fixed.  
                // Constrain child to not be bigger than us.  
                resultSize = size;                  //size为父视图大小  
                resultMode = MeasureSpec.AT_MOST;   //mode为AT_MOST  
            }  
            //2.3、子View的width或height为 WRAP_CONTENT  
            else if (childDimension == LayoutParams.WRAP_CONTENT) {  
                // Child wants to determine its own size. It can't be  
                // bigger than us.  
                resultSize = size;                  //size为父视图大小  
                resultMode = MeasureSpec.AT_MOST;   //mode为AT_MOST  
            }  
            break;  
        
        // Parent asked to see how big we want to be  
        //3、父View是UNSPECIFIED的 ！  
        case MeasureSpec.UNSPECIFIED:  
            //3.1、子View的width或height是个精确值 (an exactly size)  
            if (childDimension >= 0) {  
                // Child wants a specific size... let him have it  
                resultSize = childDimension;        //size为精确值  
                resultMode = MeasureSpec.EXACTLY;   //mode为 EXACTLY  
            }  
            //3.2、子View的width或height为 MATCH_PARENT/FILL_PARENT  
            else if (childDimension == LayoutParams.MATCH_PARENT) {  
                // Child wants to be our size... find out how big it should  
                // be  
                resultSize = 0;                        //size为0！ ,其值未定  
                resultMode = MeasureSpec.UNSPECIFIED;  //mode为 UNSPECIFIED  
            }   
            //3.3、子View的width或height为 WRAP_CONTENT  
            else if (childDimension == LayoutParams.WRAP_CONTENT) {  
                // Child wants to determine its own size.... find out how  
                // big it should be  
                resultSize = 0;                        //size为0! ，其值未定  
                resultMode = MeasureSpec.UNSPECIFIED;  //mode为 UNSPECIFIED  
            }  
            break;  
    }  
    //根据上面逻辑条件获取的mode和size构建MeasureSpec对象。  
    return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
    
}
```

每个View大小的设定都事由其父View以及该View共同决定的。但这只是一个期望的大小，每个View在测量时最终大小的设定是由setMeasuredDimension()最终决定的。因此，最终确定一个View的“测量长宽“是由以下几个方面影响：
    1、父View的MeasureSpec属性；
    2、子View的LayoutParams属性 ；
    3、setMeasuredDimension()或者其它类似设定 mMeasuredWidth 和 mMeasuredHeight 值的方法。

```
//设置View在measure过程中宽和高
protected final void setMeasuredDimension(int measuredWidth, int measuredHeight) {
    mMeasuredWidth = measuredWidth;
    mMeasuredHeight = measuredHeight;
    mPrivateFlags |= MEASURED_DIMENSION_SET;  //设置了MEASURED_DIMENSION_SET标记  
}
```


### 注：
出处：[MeasureSpec介绍及使用](http://www.cnblogs.com/nanxiaojue/p/3536381.html?utm_source=tuicool&utm_medium=referral)

## MeasureSpec 解析

#### 1、A MeasureSpec is comprised of a size and a mode.


```
/**
 * Creates a measure specification based on the supplied size and mode.
 *
 * The mode must always be one of the following:
 * <ul>
 *  <li>{@link android.view.View.MeasureSpec#UNSPECIFIED}</li>
 *  <li>{@link android.view.View.MeasureSpec#EXACTLY}</li>
 *  <li>{@link android.view.View.MeasureSpec#AT_MOST}</li>
* </ul>
*
* <p><strong>Note:</strong> On API level 17 and lower, makeMeasureSpec's
* implementation was such that the order of arguments did not matter
* and overflow in either value could impact the resulting MeasureSpec.
* {@link android.widget.RelativeLayout} was affected by this bug.
* Apps targeting API levels greater than 17 will get the fixed, more strict
* behavior.</p>
*
* @param size the size of the measure specification
* @param mode the mode of the measure specification
* @return the measure specification based on size and mode
*/
public static int makeMeasureSpec(@IntRange(from = 0, to = (1 << MeasureSpec.MODE_SHIFT) - 1) int size,
                                          @MeasureSpecMode int mode) {
    if (sUseBrokenMakeMeasureSpec) {
        return size + mode;
    } else {
        return (size & ~MODE_MASK) | (mode & MODE_MASK);
    }
}
```

```
 private static final int MODE_SHIFT = 30;
 private static final int MODE_MASK  = 0x3 << MODE_SHIFT;
```

打印成二进制：
MODE_MASK = 11000000000000000000000000000000, //0011左移动30位得到 

~MODE_MASK = 00111111111111111111111111111111； 

size & ~MODE_MASK  ：  MeasureSpec 是个32位的int型，后三十位是是分配给size的。

#### 2、mode有三种:

```
public static final int UNSPECIFIED = 0 << MODE_SHIFT;
```

//0

```
 public static final int EXACTLY    = 1 << MODE_SHIFT;
```

// 01000000000000000000000000000000


```
public static final int AT_MOST     = 2 << MODE_SHIFT;
```

// 10000000000000000000000000000000

mode & MODE_MASK  :  & 相同取1，相异取0；所以  01 & 11 = 01 ；10 & 11 = 10，后三十位全部为0

MeasureSpec 是个32位的int型，前2位是是分配给mode的。

 
MeasureSpec objects are used to push requirements down the tree from parent to child. A MeasureSpec can be in one of three modes:

* UNSPECIFIED:    The parent has not imposed any constraint on the child. It can be whatever size it wants. 无限制;，这种情况不多，一般都是父控件是AdapterView，通过measure方法传入的模式。

* EXACTLY: This is used by the parent to impose an exact size on the child. The child must use this size, and guarantee that all of its descendants will fit within this size. 限制尺寸;当我们将控件的layout_width或layout_height指定为具体数值时如andorid:layout_width="50dip"，或者为FILL_PARENT是，都是控件大小已经确定的情况，都是精确尺寸。
 
* AT MOST: This is used by the parent to impose a maximum size on the child. The child must guarantee that it and all of its descendants will fit within this size. 限制最大尺寸;，当控件的layout_width或layout_height指定为WRAP_CONTENT时，控件大小一般随着控件的子空间或内容进行变化，此时控件尺寸只要不超过父控件允许的最大尺寸即可。因此，此时的mode是AT_MOST，size给出了父控件允许的最大尺寸。

#### 3、取或：要任一表达式的一位为 1，则结果中的该位为 1。否则，结果中的该位为 0。 如下：

```
0101   （expression1）
1100   （expression2)
----
1101    (reult)
```

(size & ~MODE_MASK) | (mode & MODE_MASK)  

MeasureSpec 是个32位的int型，前2位是是分配给mode 的；后三十位是是分配给size的。

#### 4 getMode()

```
@MeasureSpecMode
public static int getMode(int measureSpec) {
    //noinspection ResourceType
    return (measureSpec & MODE_MASK);
}
```

如果现在一个 MeasureSpec = 01000000000000000000111100000000;
01000000000000000000111100000000 & MODE_MASK （11000000000000000000000000000000）= 01000000000000000000000000000000  
//结果值是EXACTLY mode

#### 5 、getSize()

```
public static int getSize(int measureSpec) {
    return (measureSpec & ~MODE_MASK);
}
```

如果现在一个 MeasureSpec = 01000000000000000000111100000000
01000000000000000000111100000000 & ~MODE_MASK（00111111111111111111111111111111）= 000000000000000000111100000000  //得出size值111100000000=3840

### 注：
出处：[MeasureSpec 解析](http://www.cnblogs.com/sueZheng/p/4046869.html)