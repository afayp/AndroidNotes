---
layout:     post
title:      "View绘制之layout过程分析"
date:       2016-07-05 18:59:25
author:     "afayp"
catalog:    true
tags:
    - Android
---



measure过程结束后，视图的大小就已经测量好了，接下来就是layout的过程了。正如其名字所描述的一样，这个方法是用于给视图进行布局的，也就是确定视图的位置。ViewRoot的performTraversals()方法会在measure结束后继续执行，并调用View的layout()方法来执行此过程，如下所示：

<!--more-->

```java
//l, t, r, b分别表示子View相对于父View的左、上、右、下的坐标
public void layout(int l, int t, int r, int b) {
        if ((mPrivateFlags3 & PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT) != 0) {
            onMeasure(mOldWidthMeasureSpec, mOldHeightMeasureSpec);
            mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
        }

        int oldL = mLeft;
        int oldT = mTop;
        int oldB = mBottom;
        int oldR = mRight;

        boolean changed = isLayoutModeOptical(mParent) ?
                setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);

        if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) {
            onLayout(changed, l, t, r, b);
            mPrivateFlags &= ~PFLAG_LAYOUT_REQUIRED;

            ListenerInfo li = mListenerInfo;
            if (li != null && li.mOnLayoutChangeListeners != null) {
                ArrayList<OnLayoutChangeListener> listenersCopy =
                        (ArrayList<OnLayoutChangeListener>)li.mOnLayoutChangeListeners.clone();
                int numListeners = listenersCopy.size();
                for (int i = 0; i < numListeners; ++i) {
                    listenersCopy.get(i).onLayoutChange(this,l,t,r,b,oldL,oldT,oldR,oldB);
                }
            }
        }

        mPrivateFlags &= ~PFLAG_FORCE_LAYOUT;
        mPrivateFlags3 |= PFLAG3_IS_LAID_OUT;
    }
```
分析：

1. 首先确定该View在其父View中的位置，请参见代码第13-14行。 
在该处调用setFrame()方法，在该方法中把l，t， r， b分别与之前的mLeft，mTop，mRight，mBottom一一作比较，假若其中任意一个值发生了变化，那么就判定该View的位置发生了变化 
2. 若View的位置发生了变化则调用onLayout()方法，请参见代码第17行

再看View的onLayout()方法：
```java
protected void onLayout(boolean changed, int left, int top, int right, int bottom) {

}
```
是个空方法，因为onLayout()用于指定子View的大小和位置，而view没有子view
那就看看ViewGroup中的onLayout()方法：
```java
protected abstract void onLayout(boolean changed,int l, int t, int r, int b);
```
是一个抽象方法,意味着ViewGroup的子类都必须重写这个方法，实现自己的逻辑。比如：FrameLayou，LinearLayout，RelativeLayout等等布局都需要重写这个方法，在该方法内依据各自的布局规则确定子View的位置。
自定义view的时候直接继承自ViewGroup可能带来的复杂处理，一般情况下，我们可以选择继承自LinearLayout，RelativeLayout等系统已有的布局从而简化这两部分的处理。

在onLayout()过程结束后，我们就可以调用getWidth()方法和getHeight()方法来获取视图的宽高了。
getWidth()方法和getMeasureWidth()方法到底有什么区别？
首先getMeasureWidth()方法在measure()过程结束后就可以获取到了，而getWidth()方法要在layout()过程结束后才能获取到。另外，getMeasureWidth()方法中的值是通过setMeasuredDimension()方法来进行设置的，而getWidth()方法中的值则是通过视图右边的坐标减去左边的坐标计算出来的。

大概流程：
ViewGroup首先调用了layout(l,t,r,b)确定了自己本身在其父View中的位置（参数一般为0,0，measure后得到的getMeasureWidth和Height，代表四个顶点的相对父布局的坐标），然后调用onLayout()确定每个子View的位置（遍历每个childView，每种ViewGroup的实现都不一样，都比较复杂），每个子View又会调用View的layout()方法来确定自己在ViewGroup的位置。 
概况地讲： 

- layout()方法用于View/ViewGroup确定自己本身在其父View的位置 
- onLayout()方法用于ViewGroup确定子View的位置
（View不需要确定子View的位置，ViewGroup才需要，所以View的onLayout为空方法）

