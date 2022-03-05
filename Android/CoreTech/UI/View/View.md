<h1 align="center">View相关问题</h1>

[toc]

## TextView

### 1. 自动换行问题，有些手机或者手表，TextView会没有满行自动换行，需要设置属性

#### 1.1 简介

自 Andriod API 23（Android 6.0）起，TextView 新增了一个 breakStrategy 属性，这个属性用于控制将一段文本分割成多行时的折行策略，通俗的讲就是决定一行到底需要显示多少文本。

breakStrategy 既可以通过 TextView 的 xml 属性 android:breakStrategy设置，也可以通过 setBreakStrategy 方法来设置。可以设置的值只有三个，它们是 android.text.Layout 类的三个常量：

- `BREAK_STRATEGY_SIMPLE`：对应 xml 属性 `"simple"`
- `BREAK_STRATEGY_HIGH_QUALITY`：对应 xml 属性 `"high_quality"`
- `BREAK_STRATEGY_BALANCED` ：对应 xml 属性 `"balanced"`

#### 1.2 三种折行策略对比

##### 1.2.1 BREAK_STRATEGY_SIMPLE

简单折行。这种策略会在每一行显示尽可能多的字符，直到这一行不能显示更多字符时才进行换行，同时这种策略下不会自动添加连词符（官方文档说，当一行只有一个单词并且宽度显示不下的情况下才会添加连词符，不过在测试过程中并没有看到连词符）。

在进行文本编辑时，后添加的文本不会影响前面文本的布局显示，比较适合可编辑的文本。EditText 默认的折行策略就是这种，因为可以避免在输入文本时由于布局刷新导致的字符跳动问题，保证用户的输入体验。

##### 1.2.2 BREAK_STRATEGY_BALANCED

平衡折行。这个策略会尽可能保证一个段落的每一行的宽度相同，必要时会添加连词符。

##### 1.2.3 BREAK_STRATEGY_HIGH_QUALITY

高质量折行。这个策略会针对整段文本的折行进行布局优化，必要时会自动添加连词符。和其他两种策略相比，这个策略会略微影响性能，并且需要更多时间进行文本布局。这个策略通常比较适合只读文本，TextView 的默认折行策略就是这种。

文字介绍没有图片来的直观，下面通过简单的例子来展示一下不同策略下段落折行情况的不同。

参考： http://www.zyiz.net/tech/detail-137242.html

## View

![](https://user-gold-cdn.xitu.io/2019/6/12/16b4a8a388f3a91a?imageslim)
ViewRoot 对应于 ViewRootImpl 类，它是连接 WindowManager 和 DecorView 的纽带，View 的三大流程均是通过 ViewRoot 来完成的。在 ActivityThread 中，当 Activity 对象被创建完毕后，会将 DecorView 添加到 Window 中，同时会创建 ViewRootImpl 对象，并将 ViewRootImpl 对象和 DecorView 建立关联

View 的整个绘制流程可以分为以下三个阶段：

- measure: 判断是否需要重新计算 View 的大小，需要的话则计算
- layout: 判断是否需要重新计算 View 的位置，需要的话则计算
- draw: 判断是否需要重新绘制 View，需要的话则重绘制

![](https://img-blog.csdn.net/20180510164327114)

## MeasureSpec

MeasureSpec表示的是一个32位的整形值，它的高2位表示测量模式SpecMode，低30位表示某种测量模式下的规格大小SpecSize。MeasureSpec 是 View 类的一个静态内部类，用来说明应该如何测量这个 View

| Mode        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| UNSPECIFIED | 不指定测量模式, 父视图没有限制子视图的大小，子视图可以是想要的任何尺寸，通常用于系统内部，应用开发中很少用到。 |
| EXACTLY     | 精确测量模式，视图宽高指定为 match_parent 或具体数值时生效，表示父视图已经决定了子视图的精确大小，这种模式下 View 的测量值就是 SpecSize 的值 |
| AT_MOST     | 最大值测量模式，当视图的宽高指定为 wrap_content 时生效，此时子视图的尺寸可以是不超过父视图允许的最大尺寸的任何尺寸 |

对于 DecorView 而言，它的MeasureSpec 由窗口尺寸和其自身的 LayoutParams 共同决定；对于普通的 View，它的 MeasureSpec 由父视图的 MeasureSpec 和其自身的 LayoutParams 共同决定

| childLayoutParams/parentSpecMode | EXACTLY             | AT_MOST             |
| -------------------------------- | ------------------- | ------------------- |
| dp/px                            | EXACTLY(childSize)  | EXACTLY(childSize)  |
| match_parent                     | EXACTLY(childSize)  | AT_MOST(parentSize) |
| wrap_content                     | AT_MOST(parentSize) | AT_MOST(parentSize) |


直接继承 View 的控件需要重写 onMeasure 方法并设置 wrap_content 时的自身大小，因为 View 在布局中使用 wrap_content，那么它的 specMode 是 AT_MOST 模式，在这种模式下，它的宽/高等于父容器当前剩余的空间大小，就相当于使用 match_parent。这解决方式如下：

```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    int widthSpecMode = MeasureSpec.getMode(widthMeasureSpec);
    int widthSpecSize = MeasureSpec.getSize(widthMeasureSpec);
    int heightSpecMode = MeasureSpec.getMode(heightMeasureSpec);
    int heightSpecSize = MeasureSpec.getSize(heightMeasureSpec);
    // 在 wrap_content 的情况下指定内部宽/高(mWidth 和 mHeight`)
    if (widthSpecMode == MeasureSpec.AT_MOST && heightSpecMode == MeasureSpec.AT_MOST) {
        setMeasuredDimension(mWidth, mHeight);
    } else if (widthSpecMode == MeasureSpec.AT_MOST) {
        setMeasureDimension(mWidth, heightSpecSize);
    } else if (heightSpecMode == MeasureSpec.AT_MOST) {
        setMeasureDimension(widthSpecSize, mHeight);
    }
}
```

## Scroller

弹性滑动对象，用于实现 View 的弹性滑动，**Scroller** 本身无法让 View 弹性滑动，需要和 View 的 ``computeScroll`` 方法配合使用。``startScroll`` 方法是无法让 View 滑动的，``invalidate`` 会导致 View 重绘，重回后会在 ``draw`` 方法中又会去调用 ``computeScroll`` 方法，``computeScroll`` 方法又会去向 Scroller 获取当前的 scrollX 和 scrollY，然后通过 ``scrollTo`` 方法实现滑动，接着又调用 ``postInvalidate`` 方法如此反复。

```java
Scroller mScroller = new Scroller(mContext);

private void smoothScrollTo(int destX) {
    int scrollX = getScrollX();
    int delta = destX - scrollX;
    // 1000ms 内滑向 destX，效果就是慢慢滑动
    mScroller.startScroll(scrollX, 0 , delta, 0, 1000);
    invalidate();
}

@Override
public void computeScroll() {
    if (mScroller.computeScrollOffset()) {
        scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
        postInvalidate();
    }
}
```

## View 的滑动

- ``scrollTo/scrollBy``  
  适合对 View 内容的滑动。``scrollBy`` 实际上也是调用了 ``scrollTo`` 方法：

```java
public void scrollTo(int x, int y) {
    if (mScrollX != x || mScrollY != y) {
        int oldX = mScrollX;
        int oldY = mScrollY;
        mScrollX = x;
        mScrollY = y;
        invalidateParentCaches();
        onScrollChanged(mScrollX, mScrollY, oldX, oldY);
        if (!awakenScrollBars()) {
            postInvalidateOnAnimation();
        }
    }
}

public void scrollBy(int x, int y) {
    scrollTo(mScrollX + x, mScrollY + y);
}
```

mScrollX的值等于 View 的左边缘和 View 内容左边缘在水平方向的距离，mScrollY的值等于 View 上边缘和 View 内容上边缘在竖直方向的距离。``scrollTo`` 和 ``scrollBy`` 只能改变 View 内容的位置而不能改变 View 在布局中的位置。

- 使用动画  
  操作简单，主要适用于没有交互的 View 和实现复杂的动画效果。
- 改变布局参数
  操作稍微复杂，适用于有交互的 View.

```java
ViewGroup.MarginLayoutParams params = (ViewGroup.MarginLayoutParams) view.getLayoutParams();
params.width += 100;
params.leftMargin += 100;
view.requestLayout();
//或者 view.setLayoutParams(params);
```

## View 的事件分发

点击事件达到顶级 View(一般是一个 ViewGroup)，会调用 ViewGroup 的 dispatchTouchEvent 方法，如果顶级 ViewGroup 拦截事件即 onInterceptTouchEvent 返回 true，则事件由 ViewGroup 处理，这时如果 ViewGroup 的 mOnTouchListener 被设置，则 onTouch 会被调用，否则 onTouchEvent 会被调用。也就是说如果都提供的话，onTouch 会屏蔽掉 onTouchEvent。在 onTouchEvent 中，如果设置了 mOnClickListenser，则 onClick 会被调用。如果顶级 ViewGroup 不拦截事件，则事件会传递给它所在的点击事件链上的子 View，这时子 View 的 dispatchTouchEvent 会被调用。如此循环。

![img](media/1-20210526174634114)



![img](media/1)

- ViewGroup 默认不拦截任何事件。ViewGroup 的 onInterceptTouchEvent 方法默认返回 false。

- View 没有 onInterceptTouchEvent 方法，一旦有点击事件传递给它，onTouchEvent 方法就会被调用。

- View 在可点击状态下，onTouchEvent 默认会消耗事件。

- ACTION_DOWN 被拦截了，onInterceptTouchEvent 方法执行一次后，就会留下记号（mFirstTouchTarget == null）那么往后的 ACTION_MOVE 和 ACTION_UP 都会拦截。`


## 在 Activity 中获取某个 View 的宽高

- Activity/View#onWindowFocusChanged

```java
// 此时View已经初始化完毕
// 当Activity的窗口得到焦点和失去焦点时均会被调用一次
// 如果频繁地进行onResume和onPause，那么onWindowFocusChanged也会被频繁地调用
public void onWindowFocusChanged(boolean hasFocus) {
    super.onWindowFocusChanged(hasFocus);
    if (hasFocus) {
        int width = view.getMeasureWidth();
        int height = view.getMeasuredHeight();
    }
}
```

- view.post(runnable)

```java
// 通过post可以将一个runnable投递到消息队列的尾部，// 然后等待Looper调用次runnable的时候，View也已经初
// 始化好了
protected void onStart() {
    super.onStart();
    view.post(new Runnable() {

        @Override
        public void run() {
            int width = view.getMeasuredWidth();
            int height = view.getMeasuredHeight();
        }
    });
}
```

- ViewTreeObserver

```java
// 当View树的状态发生改变或者View树内部的View的可见// 性发生改变时，onGlobalLayout方法将被回调
protected void onStart() {
    super.onStart();

    ViewTreeObserver observer = view.getViewTreeObserver();
    observer.addOnGlobalLayoutListener(new OnGlobalLayoutListener() {

        @SuppressWarnings("deprecation")
        @Override
        public void onGlobalLayout() {
            view.getViewTreeObserver().removeGlobalOnLayoutListener(this);
            int width = view.getMeasuredWidth();
            int height = view.getMeasuredHeight();
        }
    });
}
```

## Draw 的基本流程

```java
// 绘制基本上可以分为六个步骤
public void draw(Canvas canvas) {
    ...
    // 步骤一：绘制View的背景
    drawBackground(canvas);
    ...
    // 步骤二：如果需要的话，保持canvas的图层，为fading做准备
    saveCount = canvas.getSaveCount();
    ...
    canvas.saveLayer(left, top, right, top + length, null, flags);
    ...
    // 步骤三：绘制View的内容
    onDraw(canvas);
    ...
    // 步骤四：绘制View的子View
    dispatchDraw(canvas);
    ...
    // 步骤五：如果需要的话，绘制View的fading边缘并恢复图层
    canvas.drawRect(left, top, right, top + length, p);
    ...
    canvas.restoreToCount(saveCount);
    ...
    // 步骤六：绘制View的装饰(例如滚动条等等)
    onDrawForeground(canvas)
}
```

## 自定义 View

- 继承 View 重写 ``onDraw`` 方法

  主要用于实现一些不规则的效果，静态或者动态地显示一些不规则的图形，即重写 ``onDraw`` 方法。采用这种方式需要自己支持 wrap_content，并且 padding 也需要自己处理。

- 继承 ViewGroup 派生特殊的 Layout

  主要用于实现自定义布局，采用这种方式需要合适地处理 ViewGroup 的测量、布局两个过程，并同时处理子元素的测量和布局过程。

- 继承特定的 View

  用于扩张某种已有的View的功能

- 继承特定的 ViewGroup

  用于扩张某种已有的ViewGroup的功能

## Canvas.save()跟Canvas.restore()的调用时机

save：用来保存Canvas的状态。save之后，可以调用Canvas的平移、放缩、旋转、错切、裁剪等操作。

restore：用来恢复Canvas之前保存的状态。防止save后对Canvas执行的操作对后续的绘制有影响。

save和restore要配对使用（restore可以比save少，但不能多），如果restore调用次数比save多，会引发Error。save和restore操作执行的时机不同，就能造成绘制的图形不同。



## LinearLayout、FrameLayout、RelativeLayout性能对比，为什么？

RelativeLayout会让子View调用2次onMeasure，LinearLayout 在有weight时，也会调用子 View 2次onMeasure

RelativeLayout的子View如果高度和RelativeLayout不同，则会引发效率问题，当子View很复杂时，这个问题会更加严重。如果可以，尽量使用padding代替margin。

在不影响层级深度的情况下,使用LinearLayout和FrameLayout而不是RelativeLayout。

## 参考

* 

