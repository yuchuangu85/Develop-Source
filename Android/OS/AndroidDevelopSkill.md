<h1 align="center">Android开发技巧</h1> 

### 1.Activity切换动画过程中页面不加载或者只绘制几帧解决：
调用ViewRootImpl里的成员方法setDrawDuringWindowsAnimating--反射调用,在onAttachedToWindow或者任何view的onMeasure方法中（API19以上）

```java
try {getWindow().getDecorView().getParent().getClass().getMethod("setDrawDuringWindowsAnimating", boolean.class).                    invoke(getWindow().getDecorView().getParent(), false);		} catch (IllegalAccessException e) {			e.printStackTrace();		} catch (InvocationTargetException e) {			e.printStackTrace();		} catch (NoSuchMethodException e) {			e.printStackTrace();		}
```

```java
try {getWindow().getDecorView().getParent().getClass().getMethod("setDrawDuringWindowsAnimating", boolean.class).                    invoke(getWindow().getDecorView().getParent(), true);  } catch (IllegalAccessException e) {   e.printStackTrace();  } catch (InvocationTargetException e) {   e.printStackTrace();  } catch (NoSuchMethodException e) {   e.printStackTrace();  }
```

![](/images/Skill/skill1.jpg)

```
private boolean sInited = false;    private Method msetDrawDuringWindowsAnimatingMethod;    @Override    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {        super.onMeasure(widthMeasureSpec, heightMeasureSpec);        if (Build.VERSION.SDK_INT > Build.VERSION_CODES.JELLY_BEAN_MR2) {            Object viewRootImpl = ((Activity)getContext()).getWindow().getDecorView().getParent();            if (!sInited) {                try {                    msetDrawDuringWindowsAnimatingMethod = viewRootImpl.getClass().getMethod("setDrawDuringWindowsAnimating", boolean.class);                } catch (NoSuchMethodException e) {                    e.printStackTrace();                }                if (msetDrawDuringWindowsAnimatingMethod != null) {                    try {                        msetDrawDuringWindowsAnimatingMethod.invoke(viewRootImpl, true);                    } catch (IllegalAccessException e) {                        e.printStackTrace();                    } catch (InvocationTargetException e) {                        e.printStackTrace();                    }                }                sInited = true;            }        }    }
```

#### 2.异步加载View--优化卡顿：
只有View被添加到View树上的时候才可能会绘制，刚inflate出来的View没有被添加到View树上，所以不会进行measure和layout。measure是耗时的，如果在线程中执行，它就会减少主线程的卡顿，最后再添加到View树上，并且不用触发requestLayout。（Fragment也可以先在子线程创建，然后再UI线程添加）
![](/images/Skill/skill2.jpg)

总结：如果创建的View比较耗时，那么就在子线程去创建，例如上面代码，measure、layout都在子线程，然后在主线程去添加该View。
实例：如果列表数据需要从网络加载，且数据比较多，那么View加载相对较慢，则可以把列表的所有子View添加到子线程去创建，然后添加到列表中刷新。

#### 3.
android系统提供的GPU柱状图并不准确，无法准确的得出帧率，就利用Looper的log机制和添加全局控件来自己实现那个柱状图。
```java
getMainLooper().setMessageLogging(new Printer() {    @Override    public void println(String x) {        if(x.startsWith(">>>>> Dispatching to Handler(android.view.Choreographer$FrameHandler)")){            long time1 = System.nanoTime();            LOG.I("Frame", "帧间隔为： " + (1000000000/(time1 - time)));            time = time1;        }    }});
```

#### 4.超级权限

```java
WindowManager.LayoutParams.TYPE_APPLICATION
```

可以绕过悬浮窗权限问题，直接弹出悬浮窗。

#### 5.帮助用户打开权限

抓取手机对应型号的权限设置界面进行跳转，引导用户打开权限。

#### 6.Android7.0设置选项中有调节显示大小，如果调整到最大会报如下错误：

```java
Can't convert value at index 15 to dimension: type=0x1
```

报错原因：xml中的view尺寸没有默认值，在默认的dimens中添加一个默认值就行了。

#### 7.多进程中启动Activity的问题

不同进程中的A，B为两个Activity，都继承AppCompatActivity，A直接调用startActivity和调用startActivityForResult出现了不同的反应，就是后者不会在最近应用中留两个界面，而前者会