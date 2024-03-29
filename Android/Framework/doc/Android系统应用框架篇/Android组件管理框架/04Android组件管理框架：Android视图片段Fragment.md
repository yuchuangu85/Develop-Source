<h1 align="center">Android组件管理框架：Android视图片段Fragment</h1>

[toc]

>A Fragment is a piece of an application's user interface or behavior that can be placed in an Activity.

Fragment放置在Activity容器中，通常用来作为UI的片段，在日常的开发中也有着广泛的应用，先来看一段常用的代码。

```java
DemoFragment demoFragment = DemoFragment.newInstance("param1", "param2");
Bundle bundle = new Bundle();
demoFragment.setArguments(bundle);
getSupportFragmentManager().beginTransaction()
        .add(R.id.fragment_container, demoFragment)
        .commit();
```

这是我们非常常见的代码，借着这段代码，引出我们今天的主题：针对Fragment的全面的源码分析。

## 一 Fragment操作方法

Fragment的操作是一种事务操作，什么是事务？🤔简单来说就是一个原子操作，要么被成功执行，否则原来的操作会回滚，各个操作彼此之间互不干扰。我们先整体看下Fragment的操作
序列图。

<img src="../../../art/app/component/fragment_operation_sequence.png" height="500"/>

嗯，看起来有点长😌，不要方，我们先来看看这里面频繁出现的几个类的作用。

- FragmentActivity：这个自不必说，它是Fragment的容器Activity，只有你的Activity继承自FragmentActivity，你才能使用Fragment，Android的AppCompatActivity就继承自FragmentActivity。
- FragmentManager：Fragment的管理是由FragmentManager这个类的完成的，我们通常在Activity中使用getSupportFragmentManager()方法来获取。它是一个抽象类，其实现类是FragmentManagerImpl。
- FragmentTransaction：定义了Fragment的所有操作，我们通常通过getSupportFragmentManager().beginTransaction()方法来获取。它是一个抽象类，其实现类是BackStackRecord，BackStackRecord将Fragment、入栈信息、转场动画、相应的
操作等信息包装起来，传递给FragmentManager调用。
- FragmentHostCallback：抽象类，它将Fragment、Activity与FragmentManager串联成一个整体，FragmentActivity的内部类HostCallbacks继承了这个抽象类。
- FragmentController：它的主要职责是控制Fragment的生命周期，它在FragmentActivity里以HostCallbacks为参数被创建，持有HostCallbacks的引用。

### 1.1 操作的封装

Fragment的操作方法一共有七种：

- add
- replace
- remove
- hide
- show
- detach
- attach

```java
final class BackStackRecord extends FragmentTransaction implements
        FragmentManager.BackStackEntry, FragmentManagerImpl.OpGenerator {
    
        @Override
        public FragmentTransaction add(Fragment fragment, String tag) {
            doAddOp(0, fragment, tag, OP_ADD);
            return this;
        }
    
        @Override
        public FragmentTransaction add(int containerViewId, Fragment fragment) {
            doAddOp(containerViewId, fragment, null, OP_ADD);
            return this;
        }
    
        @Override
        public FragmentTransaction add(int containerViewId, Fragment fragment, String tag) {
            doAddOp(containerViewId, fragment, tag, OP_ADD);
            return this;
        }
        
         @Override
            public FragmentTransaction replace(int containerViewId, Fragment fragment) {
                return replace(containerViewId, fragment, null);
            }
        
            @Override
            public FragmentTransaction replace(int containerViewId, Fragment fragment, String tag) {
                if (containerViewId == 0) {
                    throw new IllegalArgumentException("Must use non-zero containerViewId");
                }
        
                doAddOp(containerViewId, fragment, tag, OP_REPLACE);
                return this;
            }
        
            @Override
            public FragmentTransaction remove(Fragment fragment) {
                Op op = new Op();
                op.cmd = OP_REMOVE;
                op.fragment = fragment;
                addOp(op);
        
                return this;
            }
        
            @Override
            public FragmentTransaction hide(Fragment fragment) {
                Op op = new Op();
                op.cmd = OP_HIDE;
                op.fragment = fragment;
                addOp(op);
        
                return this;
            }
        
            @Override
            public FragmentTransaction show(Fragment fragment) {
                Op op = new Op();
                op.cmd = OP_SHOW;
                op.fragment = fragment;
                addOp(op);
        
                return this;
            }
        
            @Override
            public FragmentTransaction detach(Fragment fragment) {
                Op op = new Op();
                op.cmd = OP_DETACH;
                op.fragment = fragment;
                addOp(op);
        
                return this;
            }
        
            @Override
            public FragmentTransaction attach(Fragment fragment) {
                Op op = new Op();
                op.cmd = OP_ATTACH;
                op.fragment = fragment;
                addOp(op);
        
                return this;
            }
}
```

你可以发现，这些方法最终都调用了addOp()方法，Op是什么？🤔Op封装了操作命令、Fragment、动画等内容。上面我们说过BackStackRecord将Fragment与相应应的操作包装起来，传递给FragmentManager调用。

```java
static final class Op {
    int cmd;
    Fragment fragment;
    int enterAnim;
    int exitAnim;
    int popEnterAnim;
    int popExitAnim;
}
```
cmd对应了响应的操作。

```java
static final int OP_NULL = 0;
static final int OP_ADD = 1;
static final int OP_REPLACE = 2;
static final int OP_REMOVE = 3;
static final int OP_HIDE = 4;
static final int OP_SHOW = 5;
static final int OP_DETACH = 6;
static final int OP_ATTACH = 7;
```

我们来看看addOp()方法的实现。

```java
final class BackStackRecord extends FragmentTransaction implements
        FragmentManager.BackStackEntry, FragmentManagerImpl.OpGenerator {
    
       ArrayList<Op> mOps = new ArrayList<>();
    
       void addOp(Op op) {
           mOps.add(op);
           op.enterAnim = mEnterAnim;
           op.exitAnim = mExitAnim;
           op.popEnterAnim = mPopEnterAnim;
           op.popExitAnim = mPopExitAnim;
       }
}
```

上面代码的最后一步是commit()方法，该方法提交事务操作，我们来看看它的实现。

```java
final class BackStackRecord extends FragmentTransaction implements
        FragmentManager.BackStackEntry, FragmentManagerImpl.OpGenerator {
    @Override
    public int commit() {
        return commitInternal(false);
    }
    
    //allowStateLoss是个标志位，表示是否允许状态丢失
    int commitInternal(boolean allowStateLoss) {
        if (mCommitted) throw new IllegalStateException("commit already called");
        if (FragmentManagerImpl.DEBUG) {
            Log.v(TAG, "Commit: " + this);
            LogWriter logw = new LogWriter(TAG);
            PrintWriter pw = new PrintWriter(logw);
            dump("  ", null, pw, null);
            pw.close();
        }
        mCommitted = true;
        if (mAddToBackStack) {
            mIndex = mManager.allocBackStackIndex(this);
        } else {
            mIndex = -1;
        }
        mManager.enqueueAction(this, allowStateLoss);
        return mIndex;
    }
}
```
可以看到BackStackRecord完成了对Fragment操作的封装，并比较给FragmentManager调用。

### 1.2 操作的调用

从上面的序列图我们可以看出，在commit()方法执行后，会调用FragmentManager.enqueueAction()方法，并通过handler.post()切换到主线程去执行这个Action，执行时间未知。
这个handler正是FragmentActivity里创建的Handler。


```java
final class BackStackRecord extends FragmentTransaction implements
        FragmentManager.BackStackEntry, FragmentManagerImpl.OpGenerator {
    
    void executeOps() {
        final int numOps = mOps.size();
        for (int opNum = 0; opNum < numOps; opNum++) {
            final Op op = mOps.get(opNum);
            final Fragment f = op.fragment;
            f.setNextTransition(mTransition, mTransitionStyle);
            //Fragment操作
            switch (op.cmd) {
                case OP_ADD:
                    f.setNextAnim(op.enterAnim);
                    mManager.addFragment(f, false);
                    break;
                case OP_REMOVE:
                    f.setNextAnim(op.exitAnim);
                    mManager.removeFragment(f);
                    break;
                case OP_HIDE:
                    f.setNextAnim(op.exitAnim);
                    mManager.hideFragment(f);
                    break;
                case OP_SHOW:
                    f.setNextAnim(op.enterAnim);
                    mManager.showFragment(f);
                    break;
                case OP_DETACH:
                    f.setNextAnim(op.exitAnim);
                    mManager.detachFragment(f);
                    break;
                case OP_ATTACH:
                    f.setNextAnim(op.enterAnim);
                    mManager.attachFragment(f);
                    break;
                default:
                    throw new IllegalArgumentException("Unknown cmd: " + op.cmd);
            }
            if (!mAllowOptimization && op.cmd != OP_ADD) {
                mManager.moveFragmentToExpectedState(f);
            }
        }
        if (!mAllowOptimization) {
            // Added fragments are added at the end to comply with prior behavior.
            mManager.moveToState(mManager.mCurState, true);
        }
    }
}
```

因而，Fragment的操作：

- add
- remove
- replace
- hide
- show
- detach
- attach

都转换成了FragmentManager的方法：

- addFragment
- removeFragment
- removeFragment + addFragment
- hideFragment
- showFragment
- detachFragment
- attachFragment

并调用FragmentManager.moveToState()方法做Fragment的状态迁移。上述的这几种Fragment的操作方法都做了哪些事情呢？🤔

- 将Fragment从mAdded列表中添加或移除。
- 改变Fragment的mAdded、mRemoving、mHidden等标志位

要理解以下方法，我们要先看看Fragment里的几个标志位的含义。

- boolean mAdded：表示Fragment是否被添加到FragmentManager里的Fragment列表mAdded中。
- boolean mRemoving：表示Fragment是否从Activity中移除。
- boolean mHidden：表示Fragment是否对用户隐藏。
- boolean mDetached：表示Fragment是否已经从宿主Activity中分离。

```java
final class FragmentManagerImpl extends FragmentManager implements LayoutInflaterFactory {
    
    //添加Fragment
   public void addFragment(Fragment fragment, boolean moveToStateNow) {
          if (mAdded == null) {
              mAdded = new ArrayList<Fragment>();
          }
          if (DEBUG) Log.v(TAG, "add: " + fragment);
          makeActive(fragment);
          if (!fragment.mDetached) {
              if (mAdded.contains(fragment)) {
                  throw new IllegalStateException("Fragment already added: " + fragment);
              }
              synchronized (mAdded) {
                  mAdded.add(fragment);
              }
              fragment.mAdded = true;
              fragment.mRemoving = false;
              if (fragment.mView == null) {
                  fragment.mHiddenChanged = false;
              }
              if (fragment.mHasMenu && fragment.mMenuVisible) {
                  mNeedMenuInvalidate = true;
              }
              if (moveToStateNow) {
                  moveToState(fragment);
              }
          }
      }
  
      //移除Fragment
      public void removeFragment(Fragment fragment) {
          if (DEBUG) Log.v(TAG, "remove: " + fragment + " nesting=" + fragment.mBackStackNesting);
          final boolean inactive = !fragment.isInBackStack();
          if (!fragment.mDetached || inactive) {
              if (mAdded != null) {
                  synchronized (mAdded) {
                      mAdded.remove(fragment);
                  }
              }
              if (fragment.mHasMenu && fragment.mMenuVisible) {
                  mNeedMenuInvalidate = true;
              }
              fragment.mAdded = false;
              fragment.mRemoving = true;
          }
      }
      
      //隐藏Fragment：将一个Fragment标记成将要隐藏状态，显示工作有completeShowHideFragment(}方法完成
      public void hideFragment(Fragment fragment) {
          if (DEBUG) Log.v(TAG, "hide: " + fragment);
          if (!fragment.mHidden) {
              fragment.mHidden = true;
              // Toggle hidden changed so that if a fragment goes through show/hide/show
              // it doesn't go through the animation.
              fragment.mHiddenChanged = !fragment.mHiddenChanged;
          }
      }
  
      //显示Fragment：将一个Fragment标记成将要显示状态，显示工作有completeShowHideFragment(}方法完成
      public void showFragment(Fragment fragment) {
          if (DEBUG) Log.v(TAG, "show: " + fragment);
          if (fragment.mHidden) {
              fragment.mHidden = false;
              // Toggle hidden changed so that if a fragment goes through show/hide/show
              // it doesn't go through the animation.
              fragment.mHiddenChanged = !fragment.mHiddenChanged;
          }
      }
  
      //将Fragment从宿主Activity分离
      public void detachFragment(Fragment fragment) {
          if (DEBUG) Log.v(TAG, "detach: " + fragment);
          if (!fragment.mDetached) {
              fragment.mDetached = true;
              if (fragment.mAdded) {
                  // We are not already in back stack, so need to remove the fragment.
                  if (mAdded != null) {
                      if (DEBUG) Log.v(TAG, "remove from detach: " + fragment);
                      synchronized (mAdded) {
                          mAdded.remove(fragment);
                      }
                  }
                  if (fragment.mHasMenu && fragment.mMenuVisible) {
                      mNeedMenuInvalidate = true;
                  }
                  fragment.mAdded = false;
              }
          }
      }
  
      //将Fragment关联3到宿主Activity
      public void attachFragment(Fragment fragment) {
          if (DEBUG) Log.v(TAG, "attach: " + fragment);
          if (fragment.mDetached) {
              fragment.mDetached = false;
              if (!fragment.mAdded) {
                  if (mAdded == null) {
                      mAdded = new ArrayList<Fragment>();
                  }
                  if (mAdded.contains(fragment)) {
                      throw new IllegalStateException("Fragment already added: " + fragment);
                  }
                  if (DEBUG) Log.v(TAG, "add from attach: " + fragment);
                  synchronized (mAdded) {
                      mAdded.add(fragment);
                  }
                  fragment.mAdded = true;
                  if (fragment.mHasMenu && fragment.mMenuVisible) {
                      mNeedMenuInvalidate = true;
                  }
              }
          }
      }
}
```

可以看到这些方法大体类似，差别在于它们处理的标志位不同，这也导致了后续的moveToState()在处理它们的时候回区别对待，具体说来：

- add操作添加一个Fragment，会依次调用 onAttach, onCreate, onCreateView, onStart and onResume 等方法。
- attach操作关联一个Fragment，会依次调用onCreateView, onStart and onResume 。
- remove操作移除一个Fragment，会依次调用nPause, onStop, onDestroyView, onDestroy and onDetach 等方法。
- detach操作分离一个Fragment，会依次调用onPause, onStop and onDestroyView  等方法。

detach后的Fragment可以再attach，而remove后的Fragment却不可以，只能重新add。

理解完了Fragment的操作，我们再来看看它的生命周期的变化，这也是我们的重点。

## Fragment生命周期

我们先来看一张完整的Fragment生命周期图。

<img src="../../../art/app/component/fragment_lifecycle_structure.png"/>

我们都知道Fragment的生命周期依赖于它的宿主Activity，但事实的情况却并不这么简单。

- onAttach：当Fragment与宿主Activity建立联系的时候调用。
- onCreate：用来完成Fragment的初始化创建工作。
- onCreateView：创建并返回View给Fragment。
- onActivityCreated：通知Fragment当前Activity的onCreate()方法已经调用完成。
- onViewStateRestored：通知Fragment以前保存的View状态都已经被恢复。
- onStart：Fragment已经对用户可见时调用，当然这个基于它的宿主Activity的onStart()方法已经被调用。
- onResume：Fragment已经开始和用户交互时调用，当然这个基于它的宿主Activity的onResume()方法已经被调用。
- onPause：Fragment不再和用户交互时调用，这通常发生在宿主Activity的onPause()方法被调用或者Fragment被修改（replace、remove）。
- onStop：Fragment不再对用户可见时调用，这通常发生在宿主Activity的onStop()方法被调用或者Fragment被修改（replace、remove）。
- onDestroyView：Fragment释放View资源时调用。
- onDetach：Fragment与宿主Activity脱离联系时调用。

在FragmentManager中，完成Fragment状态变换的主要有四个方法：

moveToState(Fragment f)
moveToState(int newState, boolean always) 
moveFragmentToExpectedState(Fragment f)
moveToState(Fragment f, int newState, int transit, int transitionStyle, boolean keepActive)

它们的触发流程也很简单，比方说FragmentActivity触发了onResume()方法。

```java
public class FragmentActivity extends BaseFragmentActivityJB implements
        ActivityCompat.OnRequestPermissionsResultCallback,
        ActivityCompatApi23.RequestPermissionsRequestCodeValidator {
    
    @Override
    protected void onDestroy() {
        super.onDestroy();

        doReallyStop(false);

        mFragments.dispatchDestroy();
        mFragments.doLoaderDestroy();
    }
}

```

它会去调用Fragment的dispatchDestory()方法，Fragment又接着会去调用FragmentManager的dispatchDestory()方法。

```java
final class FragmentManagerImpl extends FragmentManager implements LayoutInflaterFactory {
    
     public void dispatchDestroy() {
         mDestroyed = true;
         execPendingActions();
         mExecutingActions = true;
         moveToState(Fragment.INITIALIZING, false);
         mExecutingActions = false;
         mHost = null;
         mContainer = null;
         mParent = null;
     }   
}
```
最终这些处理都会回归到上面这四个方法中来，而这四个方法最终发挥作用当然是最后一个参数最多的方法，其他的方法都只是做了参数的处理和情况的判断。


Fragment定义了六种状态

```
static final int INITIALIZING = 0;     // 未创建
static final int CREATED = 1;          // 已创建
static final int ACTIVITY_CREATED = 2; // 宿主Activity已经结束创建
static final int STOPPED = 3;          // Fragment的onCreate()方法已完成，onStart()即将开始
static final int STARTED = 4;          // Fragment的onCreate()和onStart()方法都已完成，onResume()即将开始
static final int RESUMED = 5;          // Fragment的onCreate()、onStart()和onResume()方法都已完成
```

```java
final class FragmentManagerImpl extends FragmentManager implements LayoutInflaterFactory {
    
    void moveToState(Fragment f, int newState, int transit, int transitionStyle,
            boolean keepActive) {
        //状态判断
        ...
        //当前状态大于新状态，从上面的状态表可以看出，状态值越小
        //就说明处于越早的阶段，一般对应add等操作
        if (f.mState < newState) {
            ...
            switch (f.mState) {
                //未创建
                case Fragment.INITIALIZING:
                    ...
                    f.onAttach(mHost.getContext());
                    ...
                    //Fragment被定义在布局文件里的情形，需要先从布局文件里inflate出view
                    if (f.mFromLayout) {
                        f.mView = f.performCreateView(f.performGetLayoutInflater(
                                f.mSavedFragmentState), null, f.mSavedFragmentState);
                        if (f.mView != null) {
                            ...
                            f.onViewCreated(f.mView, f.mSavedFragmentState);
                        } else {
                            f.mInnerView = null;
                        }
                    }
                //已创建
                case Fragment.CREATED:
                    if (newState > Fragment.CREATED) {
                        if (!f.mFromLayout) {
                            ...
                            if (f.mView != null) {
                                ...
                                f.onViewCreated(f.mView, f.mSavedFragmentState);
                                dispatchOnFragmentViewCreated(f, f.mView, f.mSavedFragmentState,
                                        false);
                                ...
                            } else {
                                f.mInnerView = null;
                            }
                        }
                        ...
                        f.performActivityCreated(f.mSavedFragmentState);
                        ...
                    }
                case Fragment.ACTIVITY_CREATED:
                    if (newState > Fragment.ACTIVITY_CREATED) {
                        f.mState = Fragment.STOPPED;
                    }
                case Fragment.STOPPED:
                    if (newState > Fragment.STOPPED) {
                        if (DEBUG) Log.v(TAG, "moveto STARTED: " + f);
                        f.performStart();
                        dispatchOnFragmentStarted(f, false);
                    }
                case Fragment.STARTED:
                    if (newState > Fragment.STARTED) {
                        if (DEBUG) Log.v(TAG, "moveto RESUMED: " + f);
                        f.performResume();
                        dispatchOnFragmentResumed(f, false);
                        f.mSavedFragmentState = null;
                        f.mSavedViewState = null;
                    }
            }
        } 
        //当前状态大于新状态，一般对应remove等操作
        else if (f.mState > newState) {
            switch (f.mState) {
                case Fragment.RESUMED:
                    if (newState < Fragment.RESUMED) {
                        if (DEBUG) Log.v(TAG, "movefrom RESUMED: " + f);
                        f.performPause();
                        dispatchOnFragmentPaused(f, false);
                    }
                case Fragment.STARTED:
                    if (newState < Fragment.STARTED) {
                        if (DEBUG) Log.v(TAG, "movefrom STARTED: " + f);
                        f.performStop();
                        dispatchOnFragmentStopped(f, false);
                    }
                case Fragment.STOPPED:
                    if (newState < Fragment.STOPPED) {
                        if (DEBUG) Log.v(TAG, "movefrom STOPPED: " + f);
                        f.performReallyStop();
                    }
                case Fragment.ACTIVITY_CREATED:
                    if (newState < Fragment.ACTIVITY_CREATED) {
                        ...
                        f.performDestroyView();
                        dispatchOnFragmentViewDestroyed(f, false);
                        ...
                    }
                case Fragment.CREATED:
                    if (newState < Fragment.CREATED) {
                        ...
                        if (f.getAnimatingAway() != null) {
                            f.setStateAfterAnimating(newState);
                            newState = Fragment.CREATED;
                        } else {
                            ....
                            if (!f.mRetaining) {
                                f.performDestroy();
                                dispatchOnFragmentDestroyed(f, false);
                            } else {
                                f.mState = Fragment.INITIALIZING;
                            }
    
                            f.performDetach();
                            dispatchOnFragmentDetached(f, false);
                            ...
                        }
                    }
            }
        }
        ...
    }
}
```

可以发现进入该方法后会先将Fragment的当前状态与新状态进行比较：

- 如果f.mState < newState，则说明Fragment状态会从按照INITIALIZING、CREATED、ACTIVITY_CREATED、STOPPED、STARTED、RESUMED的状态进行变化，switch语句没有break，会一直顺序
执行，通知Fragment进入相应的状态，并回调Fragment里相应的生命周期方法。
- 如果f.mState < newState，则刚好和上面是反过来的过程。

这样便完成了Fragment状态的迁移和生命周期方法的回调。

## 三 Fragment回退栈

什么是Fragment回退栈呢？🤔

这个很好理解，和Activity栈相似，放在Activity里的Fragment，如果不做额外处理的话，在点击返回的时候，会直接finish当前Activity，Fragment回退栈就是用来处理Fragment返回的问题。

Fragment的回退栈也是由Fragment来管理的，关于FragmentManger的获取，一是FragmentActivity里的getSupportFragmentManager()，二是Fragment里的getChildFragmentManager()，它们
返回的都是FragmentManagerImpl对象，对Fragment的栈进行管理。

我们先来看看常用的栈操作方法。

入栈

入栈操作通过etSupportFragmentManager.beiginTransaction().addToBackStack()方法完成，它的具体实现在BackRecordStack里。

addToBackStack(String name)：入栈，这个方法的实现很简单，就是将BackRecordStack的成员变量mName赋值，mAddToBackStack置true，表示自己要添加进回退栈， 这样在调用commit()方法提交操作时，FragmentManager
会为该Fragment分配栈索引，并将它添加进回退栈列表，供后续出栈的时候调用。

出栈

出栈操作是通过getSupportFragmentManager.popBackStack()等方法完成的，它的具体实现在FragmentManagerImpl里。

- popBackStack()：栈顶Fragment出栈操作，这是一个异步方法，放在消息队列中等待执行。
- popBackStackImmediate()：栈顶Fragment出栈操作，这是一个同步方法，会被立即执行。
- popBackStack(String name, int flags)：和popBackStack()方法相似，不过指定了出栈的Fragment的name，该name以上的Fragment全部出栈，flags（POP_BACK_STACK_INCLUSIVE）用来
控制出栈的包不包括它自己。
- popBackStackImmediate(String name, int flags)：和popBackStackImmediate()方法相似，不过指定了出栈的Fragment的name，该name以上的Fragment全部出栈，flags（POP_BACK_STACK_INCLUSIVE）用来
控制出栈的包不包括它自己。
- popBackStack(String id, int flags)：和popBackStack()方法相似，不过指定了出栈的Fragment的id，该id以上的Fragment全部出栈，flags（POP_BACK_STACK_INCLUSIVE）用来
控制出栈的包不包括它自己。
- popBackStackImmediate(String id, int flags)：和popBackStackImmediate()方法相似，不过指定了出栈的Fragment的id，该id以上的Fragment全部出栈，flags（POP_BACK_STACK_INCLUSIVE）用来
控制出栈的包不包括它自己。
- getBackStackEntryCount():返回栈中Fragment的个数。
- getBackStackEntryAt(int index)：返回指定位置的Fragment。
  

我们再来看看这些方法的实现。

```java
final class FragmentManagerImpl extends FragmentManager implements LayoutInflaterFactory {

    @Override
    public void popBackStack() {
        enqueueAction(new PopBackStackState(null, -1, 0), false);
    }

    @Override
    public boolean popBackStackImmediate() {
        checkStateLoss();
        return popBackStackImmediate(null, -1, 0);
    }
}
```

PopBackStackState实现了OpGenerator接口，封装了将要出栈的Fragment的信息，包括mName、mId与mFlags信息。如果你有细心看，上面我们提到的FragmentTransaction的实现类BackStackRecord
也实现了这个接口。


```java
private class PopBackStackState implements OpGenerator {
    final String mName;
    final int mId;
    final int mFlags;

    PopBackStackState(String name, int id, int flags) {
        mName = name;
        mId = id;
        mFlags = flags;
    }

    @Override
    public boolean generateOps(ArrayList<BackStackRecord> records,
            ArrayList<Boolean> isRecordPop) {
        return popBackStackState(records, isRecordPop, mName, mId, mFlags);
    }
}
```
popBackStack()也调用了enqueueAction()方法，后续的流程和上面的Fragment操作流程是一样的，出栈操作最终对应的是Fragment的remove()操作，因此它对Fragment生命周期的影响和remove()操作相同。

至于popBackStackImmediate()的实现，则就是直接调用执行操作的方法，少了加入队列的等待过程，具体流程也和上面的Fragment操作一样。

```java
final class FragmentManagerImpl extends FragmentManager implements LayoutInflaterFactory {

    private boolean popBackStackImmediate(String name, int id, int flags) {
        execPendingActions();
        ensureExecReady(true);

        boolean executePop = popBackStackState(mTmpRecords, mTmpIsPop, name, id, flags);
        if (executePop) {
            mExecutingActions = true;
            try {
                optimizeAndExecuteOps(mTmpRecords, mTmpIsPop);
            } finally {
                cleanupExec();
            }
        }

        doPendingDeferredStart();
        burpActive();
        return executePop;
    }
}
```

























