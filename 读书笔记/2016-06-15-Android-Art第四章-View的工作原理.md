---
layout:     post
title:      "Android-Art第四章-View的工作原理"
date:       2016-06-15 17:55:20
author:     "afayp"
catalog:    true
tags:
    - Android
    - 读书笔记
---



——Android开发艺术探索 读书笔记 第四章


ViewRoot和DecorView
===


ViewRoot  
---


- ViewRoot对应ViewRootImpl类，它是连接WindowManager和DecorView的纽带。  
主要的作用有：  
A.向DecorView分发收到的用户发起的event事件，如按键，触屏，轨迹球等事件；  
B.与WindowManagerService交互，完成整个Activity的GUI的绘制。  

<!--more-->

- View的绘制流程从ViewRoot的performTraversals方法开始，经过measure(测量view的宽高)、layout（确view在父容器中的位置）和draw（将view绘制在屏幕上）三大流程。  


- 其中performMeasure方法中会调用measure方法，在measure方法中又会调用onMeasure方法，在onMeasure方法中会对所有的子元素进行measure过程，这个时候measure流程就从父容器传递到子元素了，这样就完成了一次measure过程。performLayout类似。performDraw则是在draw方法中通过dispathDraw来实现子view的遍历。


DecorView
---
- DecorView其实是一个FrameLayout，view层的事件都先经过它在传递给我们的view。其中包含了一个竖直方向的LinearLayout，上面是标题栏，下面是内容栏(id为android.R.id.content)。
- 如何得到content？	findViewById（android.R.id.content）
- 如何得到我们设置的view？	content.getChildAt(0)

<!--more-->

MeasureSpec
===
- 干什么的？ 决定一个view的尺寸规格，即测量宽高。（具体怎么决定见下面的measure过程）


- MeasureSpec将SpecMode和SpecSize打包成一个32位的int值来避免过多的对象内存分配，高2位代表SpecMode，低30位代表SpecSize，SpecMode是指测量模式，而SpecSize是指在某种测量模式下的规格大小。对应有getMode，getSize方法。


- 分三类：
	1. UNSPECIFIED：父容器不对View有任何限制，要多大给多大，这种情况一般用于系统内部，表示一种测量状态；
	2. EXACTLY：父容器已经检测出View所需要的精确大小，这个时候View的最终大小就是SpecSize所指定的值。它对应于LayoutParams中的match_parent和具体的数值这两种模式；
	3. AT_MOST：父容器指定了一个可用大小即SpecSize，View的大小不能大于这个值，具体是什么值要看不同View的具体实现。它对应于LayoutParams中的wrap_content。


- MeasureSpec和LayoutParams的对应关系：  
我们通常会给view设置LayoutParams，系统会将LayoutParams在父容器的约束下转换成对应的MeasureSpec，然后再根据这个MeasureSpec来确定View测量后的宽高。  
MeasureSpec不是唯一由LayoutParams决定的，LayoutParams需要和父容器一起才能决定view的MeasureSpec，从而进一步确定view的宽高。MeasureSpec一旦确定，在onMeasure中就可以确定view的测量宽高了。
	1. 对于DecorView，它的MeasureSpec由窗口的尺寸和其自身的LayoutParams来决定；根视图总是会充满全屏的，specSize都是等于windowSize（详细介绍在P179）
	2. 对于我们在布局中的view，它的MeasureSpec由父容器的MeasureSpec和自身的LayoutParams来共同决定。  
	

- 普通view的MeasureSpec确定主要由**getChildMeasureSpec**方法来实现，从代码来看，它与父容器的MeasureSpec和自身的LayoutParams，以及margin,padding有关。（详细图表见P182,下面是简单概括）
	1. 当view采用固定宽高时，不管父容器的MeasureSpec是什么，view的MeasureSpec都是精确模式，并且大小是LayoutParams中的大小。
	2. 当view的宽高是match_parent时，如果父容器的模式是精确模式，那么view也是精确模式，并且大小是父容器的剩余空间；如果父容器是最大模式，那么view也是最大模式，并且大小是不会超过父容器的剩余空间。
	3. 当view的宽高是wrap_content时，不管父容器的模式是精确模式还是最大模式，view的模式总是最大模式，并且大小不超过父容器的剩余空间。



View的工作流程
===
measure过程（分两种：view和viewGroup)
---


- 首先view的：  

measure是个final方法，我们无法在子类中去重写这个方法，说明Android是不允许我们改变View的measure框架。内部会调用onMeasure()方法，这里才是真正去测量并设置View大小的地方。最后调用setMeasuredDimension最终确定大小（我们一般会直接重写onMeasure，不用getDefalutSize）
 
```java
	protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
                getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
    }


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

从上面的代码可以看出，如果我们直接继承view，需要重写onMeasure方法，不然wrap_content就没用了，直接等于match_parent。解决：重写

```java
	@Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(measureWidth(widthMeasureSpec),measureHeight(heightMeasureSpec));//自定义的方法
    }

    private int measureWidth(int widthMeasureSpec) {
        int result = 0;
        int specMode = MeasureSpec.getMode(widthMeasureSpec);
        int specSize = MeasureSpec.getSize(widthMeasureSpec);

        if(specMode == MeasureSpec.EXACTLY){
            result = specSize;
        }else {
            result = 200;//这个值使我们自己定的，表示wrap_content的时候view的默认大小
            if(specMode == MeasureSpec.AT_MOST){
                result = Math.min(result,specSize);
            }
        }
        return  result;
    }

```


- viewGroup的measure过程： 

ViewGroup中定义了一个measureChildren()方法来去测量子视图的大小，如下所示：
```java
	protected void measureChildren(int widthMeasureSpec, int heightMeasureSpec) {  
	    final int size = mChildrenCount;  
	    final View[] children = mChildren;  
	    for (int i = 0; i < size; ++i) {  
	        final View child = children[i];  
	        if ((child.mViewFlags & VISIBILITY_MASK) != GONE) {  
	            measureChild(child, widthMeasureSpec, heightMeasureSpec);  
	        }  
	    }  
	}  
```

这里首先会去遍历当前布局下的所有子视图，然后逐个调用measureChild()方法来测量相应子视图的大小，如下所示：

```java
	protected void measureChild(View child, int parentWidthMeasureSpec,  
	        int parentHeightMeasureSpec) {  
	    final LayoutParams lp = child.getLayoutParams();  
	    final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,  
	            mPaddingLeft + mPaddingRight, lp.width);  
	    final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,  
	            mPaddingTop + mPaddingBottom, lp.height);  
	    child.measure(childWidthMeasureSpec, childHeightMeasureSpec);  
	}  
```

可以看到，在第4行和第6行分别调用了getChildMeasureSpec()方法来去计算子视图的MeasureSpec，计算的依据就是布局文件中定义的MATCH_PARENT、WRAP_CONTENT等值，这个方法的内部细节就不再贴出。然后在第8行调用子视图的measure()方法，并把计算出的MeasureSpec传递进去，之后的流程就和前面所介绍的一样了。

- P187有对LinearLayout的onMeasure过程的分析

- measure后可以getMeasuredWidth/Height拿到宽高，但是有些情况下系统会多次调用measure方法才能确定宽高，所以最好在onLayout中去获取宽高

- 在Activity的onCreate、onStart、onResume方法中均无法正确得到某个View的宽/高信息，这是因为View的measure过程和Activity的生命周期方法不是同步执行的，因此无法保证Activity执行了onCreate、onStart、onResume时某个View就已经测量完毕了，如果View还没有测量完毕，那么获得的宽/高就是0。  
可以通过如下四个方法来解决获取View宽/高为0的问题：  

	1.在Activity/View的onWindowFocusChanged方法（View已经初始化完毕了，宽/高已经准备好了）中获取View的宽高；  

```java
		@Override
	    public void onWindowFocusChanged(boolean hasFocus) {
	        super.onWindowFocusChanged(hasFocus);
	        if(hasFocus){
	            int width = view.getMeasuredWidth();
	            int height = view.getMeasuredHeight();
	        }
	    }
```

2.在view.post(runnable)方法（将runnable投递到消息队列的尾部，等待Looper调用此runnable的时候，View也已经初始化好了）中获取View的宽高；

```java
		@Override
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

3.使用ViewTreeObserver；

4.手动调用View的measure方法；（比较麻烦，P192）


layout过程
---


- layout方法确定view/viewGroup本身位置，onLayout方法确定所有子元素的位置。
- layout()方法接收四个参数，分别代表着左、上、右、下的坐标，当然这个坐标是相对于当前视图的父视图而言的。

		void layout(int l, int t, int r, int b)
初始化mLeft，mRight，mTop，mBottom四个值，setFrame确定四个顶点的坐标，即确定了view本身在父容器中的位置。接着调用onLayout方法。




- 一般我们只需重写onLayout方法  栗子：

```java
	@Override  
    	protected void onLayout(boolean changed, int l, int t, int r, int b) {  
        if (getChildCount() > 0) {  
            View childView = getChildAt(0);  
            childView.layout(0, 0, childView.getMeasuredWidth(), childView.getMeasuredHeight());  
        }  
    }  

```

- ### view的测量宽高和最终宽高区别（getMeasuredIWidth和getWidth）：  

1. 在View的默认实现中，View的测量宽/高和最终宽/高是相等的，只不过测量宽/高形成于View的measure过程，而最终宽/高形成于View的layout过程，即两者的赋值时机不同，测量宽高形成于measure过程，而最终宽高形成于layout过程。
	
2. 多数情况下可以认为View的测量宽/高就等于最终的宽/高，但对于在View的layout中改变了View的left、top、right、bottom四个属性时，得出的测量宽/高有可能和最终的宽/高不一致；(不过这没有意义...会导致view显示不正确)  
	
3. 还有就是View需要多次measure才确定自己测量宽/高的情况时，在前几次的测量过程中，其得出的测量宽/高有可能和最终的宽/高不一致，但最终测量宽/高还是和最终宽/高相同。



draw过程
---

- view的绘制过程通过dispatchDraw来遍历所有的子view，调用子view的draw方法。
- View的draw流程如下：

	1. 绘制背景(background.draw)；
	2. 绘制自己(onDraw)；
	3. 绘制children(dispatchDraw)；
	4. 绘制装饰(onDrawScrollBars)。  



- View有一个特殊的方法setWillNotDraw，如果一个View不需要绘制任何内容，设置这个标记位true后，系统会进行优化。默认情况下，View没有启用这个优化标记位，但是ViewGroup会默认启用这个优化标记位。这个标记位对实际开发的意义是：如果自定义控件继承于ViewGroup并且本身不具备绘制功能时，就可以开启这个标记位从而便于系统进行后续的优化。当明确知道一个ViewGroup需要通过onDraw来绘制内容时，需要显示地关闭WILL_NOT_DRAW这个标记位。


自定义view
===
分类：
---
有很多种分类方法.这里举例一种：  

1. 组合控件：将几个原生的系统控件组合在一起，比如自己的toolbar
2. 自绘控件：自己实现measure，layout，draw过程
3. 继承控件：扩展系统的控件

注意事项：
---
1. 让view支持wrap_content，及重写onMeasure方法
2. 对于直接继承view的控件，要在draw中处理padding，不然padding会无法起作用。对于直接继承viewGroup的控件要在onMeasure和onLayout中考虑padding和子元素的margin
3. 尽量不要在view中用Handler，因为view有post方法，完全可以代替Handler
4. 如果view有动画，要在onDetachedFromWindow（view不可见时调用）中停止动画和线程。对应的view的onAttachedToWindow方法。
5. view有滑动嵌套的话要处理滑动冲突