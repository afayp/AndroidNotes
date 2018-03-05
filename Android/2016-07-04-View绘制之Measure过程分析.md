---
layout:     post
title:      "View绘制之Measure过程分析"
date:       2016-07-04 17:59:25
author:     "afayp"
catalog:    true
tags:
    - Android
---


measure是测量的意思。View系统的绘制流程会从ViewRoot的`performTraversals()`方法中开始，在其内部调用View的`measure()`方法。`measure()`方法接收两个参数，`widthMeasureSpec`和`heightMeasureSpec`，这两个值分别用于确定视图的宽度和高度的规格和大小。

<!--more-->

## MeasureSpec
官方描述：
> A MeasureSpec encapsulates the layout requirements passed from parent to child.Each MeasureSpec represents a requirement for either the width or the height.A MeasureSpec is comprised of a size and a mode.

可以看出几点：`MeasureSpec`封装了父布局传递给子View的布局要求;`MeasureSpec`可以表示宽和高;`MeasureSpec`由size和mode组成。

MeasureSpec通常翻译为”测量规格”,它是一个32位的int数据。
其中高2位代表SpecMode即某种测量模式，低30位为SpecSize代表在该模式下的规格大小。可以通过如下方式获取：
```java
int specSize = MeasureSpec.getSize(measureSpec)
int specMode = MeasureSpec.getMode(measureSpec)
```
也可以通过这两个值生成新的MeasureSpec
```java
int measureSpec=MeasureSpec.makeMeasureSpec(size, mode);
```
SpecMode一共有三种: 

1. MeasureSpec.EXACTLY   	表示父视图希望子视图的大小应该是由specSize的值来决定的，系统默认会按照这个规则来设置子视图的大小，开发人员当然也可以按照自己的意愿设置成任意的大小。

2. MeasureSpec.AT_MOST 表示子视图最多只能是specSize中指定的大小，开发人员应该尽可能小得去设置这个视图，并且保证不会超过specSize。系统默认会按照这个规则来设置子视图的大小，开发人员当然也可以按照自己的意愿设置成任意的大小。
3. MeasureSpec.UNSPECIFIED 表示开发人员可以将视图按照自己的意愿设置成任意的大小，没有任何限制。这种情况比较少见，不太会用到。

每个view的measure方法中接收的`widthMeasureSpec`和`heightMeasureSpec`参数都是由父视图经过计算后传递给子视图的，说明父视图会在一定程度上决定子视图的大小。
在ViewGroup中测量子View时会调用到measureChildWithMargins()方法或者与之类似的方法。源码如下：
```java
 /**
 * @param child
 * 子View
 * @param parentWidthMeasureSpec
 * 父容器(比如LinearLayout)的宽的MeasureSpec
 * @param widthUsed
 * 父容器(比如LinearLayout)在水平方向已经占用的空间大小
 * @param parentHeightMeasureSpec
 * 父容器(比如LinearLayout)的高的MeasureSpec
 * @param heightUsed
 * 父容器(比如LinearLayout)在垂直方向已经占用的空间大小
 */
protected void measureChildWithMargins(View child, int parentWidthMeasureSpec, int widthUsed,
                                       int parentHeightMeasureSpec, int heightUsed) {
    final MarginLayoutParams lp = (ViewGroup.MarginLayoutParams) child.getLayoutParams();
    final int childWidthMeasureSpec =
              getChildMeasureSpec(parentWidthMeasureSpec, mPaddingLeft + mPaddingRight +
                                  lp.leftMargin + lp.rightMargin + widthUsed, lp.width);
    final int childHeightMeasureSpec =
              getChildMeasureSpec(parentHeightMeasureSpec, mPaddingTop + mPaddingBottom +
                                  lp.topMargin + lp.bottomMargin + heightUsed, lp.height);
    child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}
```
该方法的参数包含5个部分：要测量的子View;父容器的宽的MeasureSpec;父容器在水平方向已经占用的空间大小;父容器的高的MeasureSpec;父容器在垂直方向已经占用的空间大小。
该方法主要有四步操作： 

- 第一步： 得到子View的LayoutParams
- 第二步： 得到子View的宽的MeasureSpec
- 第三步： 得到子View的高的MeasureSpec
- 第四步： 测量子View

第二步和第三步都调用到了`getChildMeasureSpec( )`,该方法源码如下：
```java
 public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
        int specMode = View.MeasureSpec.getMode(spec);
        int specSize = View.MeasureSpec.getSize(spec);

        int size = Math.max(0, specSize - padding);

        int resultSize = 0;
        int resultMode = 0;

        switch (specMode) {
            case View.MeasureSpec.EXACTLY:
                if (childDimension >= 0) {
                    resultSize = childDimension;
                    resultMode = View.MeasureSpec.EXACTLY;
                } else if (childDimension == LayoutParams.MATCH_PARENT) {
                    resultSize = size;
                    resultMode = View.MeasureSpec.EXACTLY;
                } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                    resultSize = size;
                    resultMode = View.MeasureSpec.AT_MOST;
                }
                break;

            case View.MeasureSpec.AT_MOST:
                if (childDimension >= 0) {
                    resultSize = childDimension;
                    resultMode = View.MeasureSpec.EXACTLY;
                } else if (childDimension == LayoutParams.MATCH_PARENT) {
                    resultSize = size;
                    resultMode = View.MeasureSpec.AT_MOST;
                } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                    resultSize = size;
                    resultMode = View.MeasureSpec.AT_MOST;
                }
                break;

            case View.MeasureSpec.UNSPECIFIED:
                if (childDimension >= 0) {
                    resultSize = childDimension;
                    resultMode = View.MeasureSpec.EXACTLY;
                } else if (childDimension == LayoutParams.MATCH_PARENT) {
                    resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                    resultMode = View.MeasureSpec.UNSPECIFIED;
                } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                    resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                    resultMode = View.MeasureSpec.UNSPECIFIED;
                }
                break;
        }
        return View.MeasureSpec.makeMeasureSpec(resultSize, resultMode);
    }
```
该方法就是确定子View的MeasureSpec的具体实现。
首先看参数：

- spec：父容器(比如LinearLayout)的宽或高的MeasureSpec
- padding：父容器(比如LinearLayout)在垂直方向或者水平方向已被占用的空间
- childDimension：通过子View的LayoutParams获取到的子View的宽或高

从方法的参数可以看出：父容器(如LinearLayout)的MeasureSpec和子View的LayoutParams共同决定了子View的MeasureSpec。

再来看该方法的具体实现步骤：

- 第一步: 得到父容器的specMode和specSize，请参见第2-3行代码。

- 第二步：得到父容器在水平方向或垂直方向可用的最大空间值，请参见第5行代码。
- 第三步： 确定子View的specMode和specSize，请参见第10-50行代码。 
这一步根据父View的specMode的不同来决定子View的specMode和specSize，几种情况可以概括为这张图：
![](http://oeiu2t0ur.bkt.clouddn.com/20160510112048981.png)

至于最外层的根视图的这两个参数则是通过`performTraversals()`中的以下两个方法得到的：
```java
childWidthMeasureSpec = getRootMeasureSpec(desiredWindowWidth, lp.width);  
childHeightMeasureSpec = getRootMeasureSpec(desiredWindowHeight, lp.height); 
```
desiredWindowWidth和desiredWindowHeight等于窗体的大小；
lp.width和lp.height在创建ViewGroup实例的时候就被赋值了，它们都等于MATCH_PARENT。`getRootMeasureSpec`方法代码如下：

```java
private int getRootMeasureSpec(int windowSize, int rootDimension) {  
    int measureSpec;  
    switch (rootDimension) {  
    case ViewGroup.LayoutParams.MATCH_PARENT:  
        measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.EXACTLY);  
        break;  
    case ViewGroup.LayoutParams.WRAP_CONTENT:  
        measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.AT_MOST);  
        break;  
    default:  
        measureSpec = MeasureSpec.makeMeasureSpec(rootDimension, MeasureSpec.EXACTLY);  
        break;  
    }  
    return measureSpec;  
}  
```
使用了MeasureSpec.makeMeasureSpec()方法来组装一个MeasureSpec。当rootDimension参数等于MATCH_PARENT的时候，MeasureSpec的specMode就等于EXACTLY，当rootDimension等于WRAP_CONTENT的时候，MeasureSpec的specMode就等于AT_MOST。并且MATCH_PARENT和WRAP_CONTENT时的specSize都是等于windowSize的，也就意味着根视图总是会充满全屏的。

## View的measure()方法
源码：

```java
 public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
        boolean optical = isLayoutModeOptical(this);
        if (optical != isLayoutModeOptical(mParent)) {
            Insets insets = getOpticalInsets();
            int oWidth  = insets.left + insets.right;
            int oHeight = insets.top  + insets.bottom;
            widthMeasureSpec  = MeasureSpec.adjust(widthMeasureSpec,  optical ? -oWidth  : oWidth);
            heightMeasureSpec = MeasureSpec.adjust(heightMeasureSpec, optical ? -oHeight : oHeight);
        }

        // Suppress sign extension for the low bytes
        long key = (long) widthMeasureSpec << 32 | (long) heightMeasureSpec & 0xffffffffL;
        if (mMeasureCache == null) mMeasureCache = new LongSparseLongArray(2);

        if ((mPrivateFlags & PFLAG_FORCE_LAYOUT) == PFLAG_FORCE_LAYOUT ||
                widthMeasureSpec != mOldWidthMeasureSpec ||
                heightMeasureSpec != mOldHeightMeasureSpec) {

            // first clears the measured dimension flag
            mPrivateFlags &= ~PFLAG_MEASURED_DIMENSION_SET;

            resolveRtlPropertiesIfNeeded();

            int cacheIndex = (mPrivateFlags & PFLAG_FORCE_LAYOUT) == PFLAG_FORCE_LAYOUT ? -1 :
                    mMeasureCache.indexOfKey(key);
            if (cacheIndex < 0 || sIgnoreMeasureCache) {
                // measure ourselves, this should set the measured dimension flag back
                onMeasure(widthMeasureSpec, heightMeasureSpec);
                mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
            } else {
                long value = mMeasureCache.valueAt(cacheIndex);
                // Casting a long to int drops the high 32 bits, no mask needed
                setMeasuredDimensionRaw((int) (value >> 32), (int) value);
                mPrivateFlags3 |= PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
            }

            // flag not set, setMeasuredDimension() was not invoked, we raise
            // an exception to warn the developer
            if ((mPrivateFlags & PFLAG_MEASURED_DIMENSION_SET) != PFLAG_MEASURED_DIMENSION_SET) {
                throw new IllegalStateException("View with id " + getId() + ": "
                        + getClass().getName() + "#onMeasure() did not set the"
                        + " measured dimension by calling"
                        + " setMeasuredDimension()");
            }

            mPrivateFlags |= PFLAG_LAYOUT_REQUIRED;
        }

        mOldWidthMeasureSpec = widthMeasureSpec;
        mOldHeightMeasureSpec = heightMeasureSpec;

        mMeasureCache.put(key, ((long) mMeasuredWidth) << 32 |
                (long) mMeasuredHeight & 0xffffffffL); // suppress sign extension
    }
```
measure()这个方法是final的，因此我们无法在子类中去重写这个方法，说明Android是不允许我们改变View的measure框架的。measure方法中调用了onMeasure()方法，这里才是真正去测量并设置View大小的地方。源码如下：
```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {  
       setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),  
                            getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));  
 } 
```
主要做了三件事：

1. 在onMeasure调用setMeasuredDimension( )设置View的宽和高. 
2. 在setMeasuredDimension()中调用getDefaultSize()获取View的宽和高.
3. 在getDefaultSize()方法中又会调用到getSuggestedMinimumWidth()或者getSuggestedMinimumHeight()获取到View宽和高的最小值.

下面按照倒序分析每个方法的源码。
先来看getSuggestedMinimumWidth()
```java
//Returns the suggested minimum width that the view should use 
protected int getSuggestedMinimumWidth() {  
    return (mBackground == null) ? mMinWidth : max(mMinWidth, mBackground.getMinimumWidth());  
}
```
该方法返回View的宽度的最小值MinimumWidth.

在此需要注意该View是否有背景。

1. 若该View没有背景。 
那么该MinimumWidth为View本身的最小宽度即mMinWidth。 
有两种方法可以设置该mMinWidth值： 
第一种：XML布局文件中定义minWidth 
第二种：调用View的setMinimumWidth()方法为该值赋值 
2. 若该View有背景。 
那么该MinimumWidth为View本身最小宽度mMinWidth和View背景的最小宽度的最大值
getSuggestedMinimumHeight()方法与此处分析很类似,故不再赘述.

接下来看看getDefaultSize( )的源码
```java
public static int getDefaultSize(int size, int measureSpec) {  
     int result = size;  
     int specMode = MeasureSpec.getMode(measureSpec);  
     int specSize = MeasureSpec.getSize(measureSpec);  
     switch (specMode) {  
       case MeasureSpec.UNSPECIFIED:  
           result = size;  
           break;  
       case MeasureSpec.AT_MOST:  
       case MeasureSpec.EXACTLY:  
           result = specSize;  
           break;  
     }  
     return result;  
} 
```
该方法用于获取View的宽或者高的大小。 
该方法的第一个输入参数size就是调用getSuggestedMinimumWidth()方法获得的View的宽或高的最小值。第二个参数则是要测量view的measureSpec，里面包含着父view通过建议的宽高和模式。

从getDefaultSize()的源码里的switch可看出该方法的返回值有两种情况: 

1. measureSpec的specMode为MeasureSpec.UNSPECIFIED 
在此情况下该方法的返回值就是View的宽或者高最小值. 
该情况很少见,基本上可忽略 
2. measureSpec的specMode为MeasureSpec.AT_MOST或MeasureSpec.EXACTLY: 在此情况下getDefaultSize()的返回值就是该子View的measureSpec中的specSize。

除去第一种情况不考虑以外,可知： 
在measure阶段View的宽和高由其measureSpec中的specSize决定。

最后再来看setMeasuredDimension( )的源码：
```java
protected final void setMeasuredDimension(int measuredWidth, int measuredHeight) {  
     mMeasuredWidth = measuredWidth;  
     mMeasuredHeight = measuredHeight;  
     mPrivateFlags |= MEASURED_DIMENSION_SET;  
}
```
将测量出来的值赋给`mMeasuredWidth`和`measuredHeight`可以看出 只有Measure完之后才可以通过getMeasuredWidth()拿到view的宽度了。

系统默认的onMeasure()方法只支持EXACTLY模式，所以我们在自定义控件时如果想支持WRAP_CONTENT模式，就必须重写onMeasure（）方法。下面是常见的几种写法(不知道那种好)：

写法一（这里的mWidth和mHeight均为一个默认值；应根据具体情况而设值，比如你资源文件本来的宽高等）
```java
	protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
	    super.onMeasure(widthMeasureSpec , heightMeasureSpec);  
	    int widthSpecMode = MeasureSpec.getMode(widthMeasureSpec);  
	    int widthSpceSize = MeasureSpec.getSize(widthMeasureSpec);  
	    int heightSpecMode=MeasureSpec.getMode(heightMeasureSpec);  
	    int heightSpceSize=MeasureSpec.getSize(heightMeasureSpec);  
	 
	    if(widthSpecMode==MeasureSpec.AT_MOST&&heightSpecMode==MeasureSpec.AT_MOST){  
	        setMeasuredDimension(mWidth, mHeight);  
	    }else if(widthSpecMode==MeasureSpec.AT_MOST){  
	        setMeasuredDimension(mWidth, heightSpceSize);  
	    }else if(heightSpecMode==MeasureSpec.AT_MOST){  
	        setMeasuredDimension(widthSpceSize, mHeight);  
	    }  
	 }
```
写法二（重写onMeasure方法，创建measureWidth方法，根据测量的模式给出不同的测量值）
```java
	@Override
	protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
	    setMeasuredDimension(measureWidth(widthMeasureSpec),measureHeight(heightMeasureSpec));
	}
	
	private int measureWidth(int widthMeasureSpec) {
	    int result = 0;
	    int specMode = MeasureSpec.getMode(widthMeasureSpec);
	    int specSize = MeasureSpec.getSize(widthMeasureSpec);
	    if(specMode == MeasureSpec.EXACTLY){
	        result = specSize;
	    }else {
	        result = 200;
	        if(specMode == MeasureSpec.AT_MOST){
	            result = Math.min(result,specSize);
	        }
	    }
	
	    return result;
	}
```

几点说明：

 - 不应该出现根View的大小为wrap_content但它的一个子View大小为match_parent。情况是理论上存在的但在实际情况中是很不合理甚至错误的，我们不应该这么写。
 
分析完了View的onMeasure()源码，现在接着看ViewGroup的measure阶段的实现。
ViewGroup是一个抽象类，它没有重写View的onMeasure( )但它提供了measureChildren( )测量所有的子View。在measureChildren()方法中调用measureChild( )测量每个子View，在该方法内又会调用到child.measure( )方法，这又回到了前面熟悉的流程。 
即ViewGroup中测量子View的流程： 
measureChildren( )—>measureChild( )—>child.measure( )

measureChildren方法源码如下：
```java
protected void measureChildren(int widthMeasureSpec, int heightMeasureSpec) {
        final int size = mChildrenCount;
        final View[] children = mChildren;
        for (int i = 0; i < size; ++i) {
            final View child = children[i];
            if ((child.mViewFlags & VISIBILITY_MASK) != GONE) {
                measureChild(child, widthMeasureSpec, heightMeasureSpec);
            }
        }
    }
```
measureChild源码如下：
```java
protected void measureChild(View child, int parentWidthMeasureSpec,
            int parentHeightMeasureSpec) {
        final LayoutParams lp = child.getLayoutParams();

        final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                mPaddingLeft + mPaddingRight, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                mPaddingTop + mPaddingBottom, lp.height);

        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }
```
这里的measureChild( )和前面的measureChildWithMargins( )有什么相同点和区别呢？

- measureChildWithMargins( )和measureChild( )均用来测量子View的大小
- 两者在调用getChildMeasureSpec( )均要计算父View已占空间
- 在measureChild( )计算父View所占空间为mPaddingLeft + mPaddingRight，即父容器左右两侧的padding值之和
- measureChildWithMargins( )计算父View所占空间为mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin+ widthUsed。此处，除了父容器左右两侧的padding值之和还包括了子View左右的margin值之和( lp.leftMargin + lp.rightMargin)，因为这两部分也是不能用来摆放子View的应算作父View已经占用的空间。这点从方法名measureChildWithMargins也可以看出来它是考虑了子View的margin所占空间的。
- 所以measureChild和measureChildWithMargins的区别就是是否把margin也作为子视图的大小

measure过程可以概括为下图：
![](http://oeiu2t0ur.bkt.clouddn.com/20162020202.png)

注意点：

 - `onMeasure(int widthMeasureSpec, int heightMeasureSpec)`这个方法里面的参数和`measure（）`是一样的，我们可以通过`MeasureSpec.getSize（）/MeasureSpec.getMode`等方法拿出父控件为我们初步测量好的宽高(在measureChild中已测量好)，然后根据Mode按照我们的需求决定view最后的宽高。
 - `setMeasuredDimension`之后view的大小可能还会发生变化，因为父view可能还会调整我们子view的大小，（这个好像我们也没办法，但是很少遇到）。我们可以在`onSizeChanged`中拿到最后真正确定的大小。(一般获取宽高都在onSizeChanged中拿最好)