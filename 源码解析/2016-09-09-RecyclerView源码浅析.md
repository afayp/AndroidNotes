---
layout:     post
title:      "RecyclerView源码浅析"
date:       2016-09-09 19:50:30
author:     "afayp"
catalog:    true
tags:
    - Android
    - 源码解析
---

# RecyclerView源码浅析

`RecyclerView`的基本使用只需要提供一个`RecyclerView.Apdater`的实现用于处理数据集与`ItemView`的绑定关系，和一个`RecyclerView.LayoutManager`的实现用于测量并布局` ItemView`。

<!--more-->

# 绘制流程
`android`控件的绘制可以分为3个步骤：`measure`、`layout`、`draw`。`RecyclerView`也不例外，`RecyclerView`将它的`measure`与`layout`过程委托给了`RecyclerView.LayoutManager`来处理，并且，它对子控件的`measure`及`layout`过程是逐个处理的，也就是说，执行完成一个子控件的`measure`及`layout`过程再去执行下一个。看下`RecyclerView`的`onMeasure`方法：

```java
protected void onMeasure(int widthSpec, int heightSpec) {
    ...
    if (mLayout.mAutoMeasure) {
        final int widthMode = MeasureSpec.getMode(widthSpec);
        final int heightMode = MeasureSpec.getMode(heightSpec);
        final boolean skipMeasure = widthMode == MeasureSpec.EXACTLY
                && heightMode == MeasureSpec.EXACTLY;
        mLayout.onMeasure(mRecycler, mState, widthSpec, heightSpec);
        if (skipMeasure || mAdapter == null) {
            return;
        }
        ...
        dispatchLayoutStep2();

        mLayout.setMeasuredDimensionFromChildren(widthSpec, heightSpec);
        ...
    } else {
        ...
    }
}
```
再看下`dispatchLayoutStep2()`方法：
```
private void dispatchLayoutStep2() {
    ...
    mLayout.onLayoutChildren(mRecycler, mState);
    ...
}
```
上面`的mLayout`就是一个`RecyclerView.LayoutManager`实例,android提供了3个`RecyclerView.LayoutManager`的默认实现，以`LinearLayoutManager`为例，看下它的`onLayoutChildren`方法：
```java
public void onLayoutChildren(RecyclerView.Recycler recycler, RecyclerView.State state) {
    // layout algorithm:
    // 1) by checking children and other variables, find an anchor coordinate and an anchor
    //  item position.
    // 2) fill towards start, stacking from bottom
    // 3) fill towards end, stacking from top
    // 4) scroll to fulfill requirements like stack from bottom.
    ...
    mAnchorInfo.mLayoutFromEnd = mShouldReverseLayout ^ mStackFromEnd;
    // calculate anchor position and coordinate
    updateAnchorInfoForLayout(recycler, state, mAnchorInfo);
    ...
    if (mAnchorInfo.mLayoutFromEnd) {
        ...
    } else {
        // fill towards end
        updateLayoutStateToFillEnd(mAnchorInfo);
        mLayoutState.mExtra = extraForEnd;
        fill(recycler, mLayoutState, state, false);
        endOffset = mLayoutState.mOffset;
        final int lastElement = mLayoutState.mCurrentPosition;
        if (mLayoutState.mAvailable > 0) {
            extraForStart += mLayoutState.mAvailable;
        }
        // fill towards start
        updateLayoutStateToFillStart(mAnchorInfo);
        mLayoutState.mExtra = extraForStart;
        mLayoutState.mCurrentPosition += mLayoutState.mItemDirection;
        fill(recycler, mLayoutState, state, false);
        startOffset = mLayoutState.mOffset;
        ...
    }
    ...
}
```
`mAnchorInfo`为布局锚点信息，包含了子控件在Y轴上起始绘制偏移量（coordinate），`ItemView`在Adapter中的索引位置（position）和布局方向（mLayoutFromEnd）——这里是指start、end方向。
这部分代码的功能就是：确定布局锚点，以此为起点向开始和结束方向填充`ItemView`，如图所示： 
![](http://oeiu2t0ur.bkt.clouddn.com/path5784-0.png)

`fill()`方法的作用就是填充`ItemView`，现在来看下`fill()`方法：
```java
int fill(RecyclerView.Recycler recycler, LayoutState layoutState,
        RecyclerView.State state, boolean stopOnFocusable) {
    ...
    int remainingSpace = layoutState.mAvailable + layoutState.mExtra;
    LayoutChunkResult layoutChunkResult = new LayoutChunkResult();
    while (...&&layoutState.hasMore(state)) {
        ...
        layoutChunk(recycler, state, layoutState, layoutChunkResult);

        ...
        if (...) {
            layoutState.mAvailable -= layoutChunkResult.mConsumed;
            remainingSpace -= layoutChunkResult.mConsumed;
        }
        if (layoutState.mScrollingOffset != LayoutState.SCOLLING_OFFSET_NaN) {
            layoutState.mScrollingOffset += layoutChunkResult.mConsumed;
            if (layoutState.mAvailable < 0) {
                layoutState.mScrollingOffset += layoutState.mAvailable;
            }
            recycleByLayoutState(recycler, layoutState);
        }
    }
    ...
}
```
下面是`layoutChunk()`方法：
```java
void layoutChunk(RecyclerView.Recycler recycler, RecyclerView.State state,
        LayoutState layoutState, LayoutChunkResult result) {
    View view = layoutState.next(recycler);
    ...
    if (layoutState.mScrapList == null) {
        if (mShouldReverseLayout == (layoutState.mLayoutDirection
                == LayoutState.LAYOUT_START)) {
            addView(view);
        } else {
            addView(view, 0);
        }
    }
    ...
    measureChildWithMargins(view, 0, 0);
    ...
    // We calculate everything with View's bounding box (which includes decor and margins)
    // To calculate correct layout position, we subtract margins.
    layoutDecorated(view, left + params.leftMargin, top + params.topMargin,
            right - params.rightMargin, bottom - params.bottomMargin);
    ...
}
```

这里的`addView()`方法，其实就是`ViewGroup`的`addView()`方法；`measureChildWithMargins()`方法看名字就知道是用于测量子控件大小的，下面是`layoutDecoreated()`方法：
```java
public void layoutDecorated(...) {
        ...
        child.layout(...);
}
```
总结上面代码，在`RecyclerView`的`measure`及`layout`阶段，填充`ItemView`的算法为：向父容器增加子控件，测量子控件大小，布局子控件，布局锚点向当前布局方向平移子控件大小，重复上诉步骤至`RecyclerView`可绘制空间消耗完毕或子控件已全部填充。 

这样所有的子控件的`measure`及`layout`过程就完成了。回到`RecyclerView`的`onMeasure`方法，执行`mLayout.setMeasuredDimensionFromChildren(widthSpec, heightSpec)`这行代码的作用就是根据子控件的大小，设置`RecyclerView`的大小。至此，`RecyclerView`的`measure`和`layout`实际上已经完成了。 
　　
特别说明的是在23.2.0版本之前，`RecyclerView`是不支持`WRAP_CONTENT`的
先看下`RecyclerView`的`onLayout()`方法：
```java
protected void onLayout(boolean changed, int l, int t, int r, int b) {
    ...
    dispatchLayout();
    ...
}
```
这是dispatchLayout()方法：
```java
void dispatchLayout() {
    ...
    if (mState.mLayoutStep == State.STEP_START) {
        dispatchLayoutStep1();
        ...
        dispatchLayoutStep2();
    }
    dispatchLayoutStep3();
    ...
}
```
可以看出，这里也会执行子控件的`measure`及`layout`过程。结合`onMeasure`方法对`skipMeasure`的判断可以看出，如果要支持`WRAP_CONTENT`，那么子控件的`measure`及`layout`就会提前在RecyclerView的测量方法中执行完成，也就是说，先确定了子控件的大小及位置后，再由此设置`RecyclerView`的大小；如果是其它情况(测量模式为EXACTLY)，子控件的`measure`及`layout`过程就会延迟至`RecyclerView`的`layout`过程（RecyclerView.onLayout()）中执行。

以上就是`RecyclerView`的`measure`及`layout`过程，下面来看下它的`draw`过程。 
`RecyclerView`的`draw`过程可以分为２部分来看：`RecyclerView`负责绘制所有`decoration`；`ItemView`的绘制由`ViewGroup`处理，这里的绘制是android常规绘制逻辑，本文就不再阐述了。下面来看看`RecyclerView的draw()`和`onDraw()`方法：
```java
@Override
public void draw(Canvas c) {
    super.draw(c);

    final int count = mItemDecorations.size();
    for (int i = 0; i < count; i++) {
        mItemDecorations.get(i).onDrawOver(c, this, mState);
    }
    ...
}

@Override
public void onDraw(Canvas c) {
    super.onDraw(c);

    final int count = mItemDecorations.size();
    for (int i = 0; i < count; i++) {
        mItemDecorations.get(i).onDraw(c, this, mState);
    }
}
```

前面`itemview`的测量会调用`measureChildWithMargins()`方法：
```java
public void measureChildWithMargins(View child, int widthUsed, int heightUsed) {
        final LayoutParams lp = (LayoutParams) child.getLayoutParams();

        final Rect insets = mRecyclerView.getItemDecorInsetsForChild(child);
        widthUsed += insets.left + insets.right;
        heightUsed += insets.top + insets.bottom;

        final int widthSpec = ...
        final int heightSpec = ...
        if (shouldMeasureChild(child, widthSpec, heightSpec, lp)) {
            child.measure(widthSpec, heightSpec);
        }
    }
```
通过`getItemDecorInsetsForChild()`方法获取分割线的大小（分割线实际上就是一个矩形）：
```java
 Rect getItemDecorInsetsForChild(View child) {
    ...
    final Rect insets = lp.mDecorInsets;
    insets.set(0, 0, 0, 0);
    final int decorCount = mItemDecorations.size();
    for (int i = 0; i < decorCount; i++) {
        mTempRect.set(0, 0, 0, 0);
        mItemDecorations.get(i).getItemOffsets(mTempRect, child, this, mState);
        insets.left += mTempRect.left;
        insets.top += mTempRect.top;
        insets.right += mTempRect.right;
        insets.bottom += mTempRect.bottom;
    }
    lp.mInsetsDirty = false;
    return insets;
}
```

方法`getItemOffsets()`就是我们在实现一个`RecyclerView.ItemDecoration`时可以重写的方法，通过`mTempRect`的大小，可以为每个`ItemView`设置位置偏移量，这个偏移量最终会参与计算`ItemView`的大小，也就是说`ItemView`的大小是包含这个位置偏移量的。我们在重写`getItemOffsets()`时，可以指定任意数值的偏移量：
![](http://oeiu2t0ur.bkt.clouddn.com/text9449-1.png)

4个方向的位置偏移量对`应mTempRect`的4个属性(`left,top,right,bottom`)，我以`top offset`的值在垂直线性布局中的应用来举例说明下。如果top offset等于0，那么ItemView之间就没有空隙；如果top offset大于0，那么ItemView之前就会有一个间隙；如果top offset小于0，那么ItemView之间就会有重叠的区域。 

# 滑动
`RecyclerView`的滑动过程可以分为2个阶段：
> 手指在屏幕上移动，使RecyclerView滑动的过程，可以称为scroll；
手指离开屏幕，RecyclerView继续滑动一段距离的过程，可以称为fling。

现在先看看`RecyclerView`的触屏事件处理`onTouchEvent()`方法：
```java
public boolean onTouchEvent(MotionEvent e) {
    ...
    if (mVelocityTracker == null) {
        mVelocityTracker = VelocityTracker.obtain();
    }
    ...
    switch (action) {
        ...
        case MotionEvent.ACTION_MOVE: {
            ...
            final int x = (int) (MotionEventCompat.getX(e, index) + 0.5f);
            final int y = (int) (MotionEventCompat.getY(e, index) + 0.5f);
            int dx = mLastTouchX - x;
            int dy = mLastTouchY - y;
            ...
            if (mScrollState != SCROLL_STATE_DRAGGING) {
                ...
                if (canScrollVertically && Math.abs(dy) > mTouchSlop) {
                    if (dy > 0) {
                        dy -= mTouchSlop;
                    } else {
                        dy += mTouchSlop;
                    }
                    startScroll = true;
                }
                if (startScroll) {
                    setScrollState(SCROLL_STATE_DRAGGING);
                }
            }

            if (mScrollState == SCROLL_STATE_DRAGGING) {
                mLastTouchX = x - mScrollOffset[0];
                mLastTouchY = y - mScrollOffset[1];

                if (scrollByInternal(
                        canScrollHorizontally ? dx : 0,
                        canScrollVertically ? dy : 0,
                        vtev)) {
                    getParent().requestDisallowInterceptTouchEvent(true);
                }
            }
        } break;
        ...
        case MotionEvent.ACTION_UP: {
            ...
            final float yvel = canScrollVertically ?
                    -VelocityTrackerCompat.getYVelocity(mVelocityTracker, mScrollPointerId) : 0;
            if (!((xvel != 0 || yvel != 0) && fling((int) xvel, (int) yvel))) {
                setScrollState(SCROLL_STATE_IDLE);
            }
            resetTouch();
        } break;
        ...
    }
    ...
}
```

以垂直方向的滑动来说明。当`RecyclerView`接收到`ACTION_MOVE`事件后，会先计算出手指移动距离（dy），并与滑动阀值（`mTouchSlop`）比较，当大于此阀值时将滑动状态设置为`SCROLL_STATE_DRAGGING`，而后调用`scrollByInternal()`方法，使`RecyclerView`滑动，这样RecyclerView的滑动的第一阶段`scroll`就完成了；
当接收到`ACTION_UP`事件时，会根据之前的滑动距离与时间计算出一个初速度`yvel`，这步计算是由`VelocityTracker`实现的，然后再以此初速度，调用方法`fling()`，完成RecyclerView滑动的第二阶段fling。

# Recycler
`Recycler`的作用就是重用`ItemView`。在填充`ItemView`的时候，`ItemView`是从它获取的；滑出屏幕的`ItemView`是由它回收的。对于不同状态的`ItemView`存储在了不同的集合中，比如有`scrapped、cached、exCached、recycled`。
回到之前的`layoutChunk`方法中，有行代码`layoutState.next(recycler)`，它的作用自然就是获取`ItemView`，我们进入这个方法查看，最终它会调用到`RecyclerView.Recycler.getViewForPosition()`方法：
```java
View getViewForPosition(int position, boolean dryRun) {
    ...
    // 0) If there is a changed scrap, try to find from there
    if (mState.isPreLayout()) {
        holder = getChangedScrapViewForPosition(position);
        fromScrap = holder != null;
    }
    // 1) Find from scrap by position
    if (holder == null) {
        holder = getScrapViewForPosition(position, INVALID_TYPE, dryRun);
        ...
    }
    if (holder == null) {
        ...
        // 2) Find from scrap via stable ids, if exists
        if (mAdapter.hasStableIds()) {
            holder = getScrapViewForId(mAdapter.getItemId(offsetPosition), type, dryRun);
            ...
        }
        if (holder == null && mViewCacheExtension != null) {
            final View view = mViewCacheExtension
                    .getViewForPositionAndType(this, position, type);
            if (view != null) {
                holder = getChildViewHolder(view);
                ...
            }
        }
        if (holder == null) {
            ...
            holder = getRecycledViewPool().getRecycledView(type);
            ...
        }
        if (holder == null) {
            holder = mAdapter.createViewHolder(RecyclerView.this, type);
            ...
        }
    }
    ...
    boolean bound = false;
    if (mState.isPreLayout() && holder.isBound()) {
        // do not update unless we absolutely have to.
        holder.mPreLayoutPosition = position;
    } else if (!holder.isBound() || holder.needsUpdate() || holder.isInvalid()) {
        ...
        mAdapter.bindViewHolder(holder, offsetPosition);
        ...
    }
    ...
}
```
这个方法比较长，我先解释下它的逻辑吧。根据列表位置获取`ItemView`，先后从`scrapped、cached、exCached、recycled`集合中查找相应的`ItemView`，如果没有找到，就创建（`Adapter.createViewHolder()`），最后与数据集绑定。其中`scrapped`、`cached`和`exCached`集合定义在`RecyclerView.Recycler`中，分别表示将要在`RecyclerView`中删除的`ItemView`、一级缓存`ItemView`和二级缓存`ItemView`，`cached`集合的大小默认为２，`exCached`是需要我们通过`RecyclerView.ViewCacheExtension`自己实现的，默认没有；`recycled`集合其实是一个`Map`，定义在`RecyclerView.RecycledViewPool`中，将`ItemView以ItemTyp`e分类保存了下来，这里算是`RecyclerView`设计上的亮点，通过`RecyclerView.RecycledViewPool`可以实现在不同的`RecyclerView`之间共享`ItemView`，只要为这些不同`RecyclerView`设置同一个`RecyclerView.RecycledViewPool`就可以了。 

上面解释了`ItemView`从不同集合中获取的方式，那么`RecyclerView`又是在什么时候向这些集合中添加`ItemView`的呢？下面我逐个介绍下。 

`scrapped`集合中存储的其实是正在执行`REMOVE`操作的`ItemView`，这部分会在后文进一步描述。 
在`fill()`方法的循环体中有行代码`recycleByLayoutState(recycler, layoutState)`;，最终这个方法会执行到`RecyclerView.Recycler.recycleViewHolderInternal()`方法：
```java
void recycleViewHolderInternal(ViewHolder holder) {
        ...
        if (forceRecycle || holder.isRecyclable()) {
            if (!holder.hasAnyOfTheFlags(ViewHolder.FLAG_INVALID | ViewHolder.FLAG_REMOVED
                    | ViewHolder.FLAG_UPDATE)) {
                // Retire oldest cached view
                final int cachedViewSize = mCachedViews.size();
                if (cachedViewSize == mViewCacheMax && cachedViewSize > 0) {
                    recycleCachedViewAt(0);
                }
                if (cachedViewSize < mViewCacheMax) {
                    mCachedViews.add(holder);
                    cached = true;
                }
            }
            if (!cached) {
                addViewHolderToRecycledViewPool(holder);
                recycled = true;
            }
        }
        ...
    }
```
这个方法的逻辑是这样的：首先判断集合`cached`是否満了，如果已満就从`cached`集合中移出一个到`recycled`集合中去，再把新的`ItemView`添加到`cached`集合；如果不満就将`ItemView`直接添加到`cached`集合。 
最后`exCached`集合是我们自己创建的，所以添加删除元素也要我们自己实现。

# ViewHolder
对于传统的`AdapterView`，需要在实现的`Adapter`类中手动加`ViewHolder`，`RecyclerView`直接将`ViewHolder`内置，并在原来基础上功能上更强大。 
`ViewHolder`描述`RecylerView`中某个位置的`itemView`和元数据信息，属于`Adapter`的一部分。其实现类通常用于保存`findViewById`的结果。 主要元素组成有：
```java
public static abstract class ViewHolder {
    View itemView;//itemView
    int mPosition;//位置
    int mOldPosition;//上一次的位置
    long mItemId;//itemId
    int mItemViewType;//itemViewType
    int mPreLayoutPosition;
    int mFlags;//ViewHolder的状态标志
    int mIsRecyclableCount;
    Recycler mScrapContainer;//若非空，表明当前ViewHolder对应的itemView可以复用
｝
```
主要介绍下`mFlags`:
> 
- FLAG_BOUND——ViewHolder已经绑定到某个位置，mPosition、mItemId、mItemViewType都有效 
- FLAG_UPDATE——ViewHolder绑定的View对应的数据过时需要重新绑定，mPosition、mItemId还是一致的 
- FLAG_INVALID——ViewHolder绑定的View对应的数据无效，需要完全重新绑定不同的数据 
- FLAG_REMOVED——ViewHolder对应的数据已经从数据集移除 
- FLAG_NOT_RECYCLABLE——ViewHolder不能复用 
- FLAG_RETURNED_FROM_SCRAP——这个状态的ViewHolder会加到scrap list被复用。 
- FLAG_CHANGED——ViewHolder内容发生变化，通常用于表明有ItemAnimator动画 
- FLAG_IGNORE——ViewHolder完全由LayoutManager管理，不能复用 
- FLAG_TMP_DETACHED——ViewHolder从父RecyclerView临时分离的标志，便于后续移除或添加回来 
- FLAG_ADAPTER_POSITION_UNKNOWN——ViewHolder不知道对应的Adapter的位置，直到绑定到一个新位置 
- FLAG_ADAPTER_FULLUPDATE——方法addChangePayload(null)调用时设置

# 数据集、动画

`RecyclerView`定义了4种针对数据集的操作，分别是`ADD、REMOVE、UPDATE、MOVE`，封装在了`AdapterHelper.UpdateOp`类中，并且所有操作由一个大小为30的对象池管理着。当我们要对数据集作任何操作时，都会从这个对象池中取出一个`UpdateOp`对象，放入一个等待队列中，最后调用`RecyclerView.RecyclerViewDataObserver.triggerUpdateProcessor()`方法，根据这个等待队列中的信息，对所有子控件重新测量、布局并绘制且执行动画。以上就是我们调用`Adapter.notifyItemXXX()`系列方法后发生的事。 

# 参考
[RecyclerView源码分析](http://mouxuejie.com/blog/2016-03-06/recyclerview-analysis/)
[RecyclerView剖析](http://blog.csdn.net/qq_23012315/article/details/50807224)
[读源码-用设计模式解析RecyclerView](http://www.jianshu.com/p/c82cebc4e798)
[RecyclerView技术栈](http://www.jianshu.com/p/16712681731e)
[RecyclerView优秀文集](https://github.com/CymChad/CymChad.github.io)