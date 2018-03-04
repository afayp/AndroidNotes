---
layout:     post
title:      "Android-Art第十二章-Bitmap的加载和Cache"
date:       2016-06-29 23:48:11
author:     "afayp"
catalog:    true
tags:
    - Android
    - 读书笔记
---



## 12.1 Bitmap的高速加载

### Bitmap是如何加载的
`BitmapFactory`类提供了四类方法：`decodeFile、decodeResource、decodeStream和decodeByteArray`从不同来源加载出一个`Bitmap`对象，分别对应文件，资源，输入流以及字节数据，这四个方法最终是在Android底层实现的，对应`BitmapFactory`的几个`native`方法。

<!--more-->


### 如何高效加载Bitmap
采用`BitmapFactory.Options`按照一定的采样率来加载所需尺寸的图片，因为`imageview`所需的图片大小往往小于图片的原始尺寸。

`BitmapFactory.Options`的`inSampleSize`参数，即采样率，为k，则宽高均为原来1/k,像素数和内存大小变为1/(k*k),建议k为2的指数。
将`Options`的`inJustDecodeBounds`设置为`true`的时候，BitmapFactory只会解析图片的原始宽高信息，并不会真正的加载图片，所以这个操作是轻量级的。需要注意的是，这个时候`BitmapFactory`获取的图片宽高信息和图片的位置（放在不同drawable目录下）以及程序运行的设备有关，这都会导致`BitmapFactory`获取到不同的结果，这和Android资源加载机制有关。

### 如何获取采样率
下面是一个常用的缩放图片的方法，返回的是缩放后的Bitmap:
```java
	public Bitmap decodeSampledBitmapFromResource(Resources res, int resId, int reqWidth, int reqHeight) {
	    // First decode with inJustDecodeBounds=true to check dimensions
	    final BitmapFactory.Options options = new BitmapFactory.Options();
	    options.inJustDecodeBounds = true;
	    BitmapFactory.decodeResource(res, resId, options);
	 
	    // Calculate inSampleSize
	    options.inSampleSize = calculateInSampleSize(options, reqWidth, reqHeight);
	 
	    // Decode bitmap with inSampleSize set
	    options.inJustDecodeBounds = false;
	    return BitmapFactory.decodeResource(res, resId, options);
	}
	 
	public int calculateInSampleSize(BitmapFactory.Options options, int reqWidth, int reqHeight) {
	    if (reqWidth == 0 || reqHeight == 0) {
	        return 1;
	    }
	 
	    // Raw height and width of image
	    final int height = options.outHeight;
	    final int width = options.outWidth;
	    Log.d(TAG, "origin, w= " + width + " h=" + height);
	    int inSampleSize = 1;
	 
	    if (height > reqHeight || width > reqWidth) {
	        final int halfHeight = height / 2;
	        final int halfWidth = width / 2;
	 
	        // Calculate the largest inSampleSize value that is a power of 2 and
	        // keeps both height and width larger than the requested height and width.
	        while ((halfHeight / inSampleSize) >= reqHeight && (halfWidth / inSampleSize) >= reqWidth) {
	            inSampleSize *= 2;
	        }
	    }
	 
	    Log.d(TAG, "sampleSize:" + inSampleSize);
	    return inSampleSize;
	}
```	
要注意的是，`BitmapFactory`的四个方法都是支持采样加载的（即设置Options参数） ，但是`decodeStream`比较特殊。

如果你拿到输入流后调用： `BitmapFactory.decodeStream(fileInputStream,null,options)`会有问题。
原因是`FileInputStream`是一种有序的文件流，而两次`decodeStream`调用会影响文件流的位置属性，导致第二次`decodeStream`时得到的是null（反正就是不行。。）。

解决方法：用`BitmapFactory.decodeFileDescriptor`方法来加载：
```java
   FileInputStream fis = (FileInputStream) snapShot.getInputStream(0);
   BitmapFactory.Options options = new BitmapFactory.Options();
   options.inSampleSize = 2;//计算压缩比
   Bitmap bitmap = BitmapFactory.decodeFileDescriptor(fis.getFD(), null, options);//加载一张压缩后的图片
```	

## 12.2 Android中的缓存策略
最常用的缓存算法是`LRU`，核心是当缓存满时，会优先淘汰那些近期最少使用的缓存对象，系统中采用`LRU`算法的缓存有两种：`LruCache`(内存缓存)和`DiskLruCache`(磁盘缓存)。两者的详细用法见 [Android中的图片缓存(LruCache和DiskLruCache)](http://afayp.me/2016/06/03/%E5%9B%BE%E7%89%87%E7%BC%93%E5%AD%98/)，不赘述。

关于几种引用
> 
	• 强引用：直接的对象引用
	• 软引用：当一个对象只有软引用时，系统内存不足时，此对象会被回收
	• 弱引用：当对象只有弱引用时，此对象随时会被gc回收

ImageLoader的实现具体内容看[源码](https://github.com/singwhatiwanna/android-art-res/blob/master/Chapter_12/src/com/ryg/chapter_12/loader/ImageLoader.java)
功能：图片的同步/异步加载，图片压缩，内存缓存，磁盘缓存，网络拉取

## 12.3 ImageLoader的使用

### 错位问题：

#### 列表item错位原因
在`ListView、RecyclerView、GridView`中会用到`View`的复用，假设一个`itemA`正在从网络加载图片，它对应的`imageView`为A，这个时候用户下滑列表，`itemB`复用了`imageView A`，等图片下载完了如果直接给`imageViewA`设置这个图片的话，结果就是`itemB`显示了`itemA`要显示的图片

#### 解决方法
给显示图片的`imageview`添加`tag`属性，值为要加载的图片的目标`url`，显示的时候判断一下从`imageView`取出的`url`是否是这个`item`要加载的`url`，当然如果你用`Glide`等工具来加载图片就不需要考虑这个问题了，它们已经帮你解决了。

### 优化列表的卡顿现象

- 不要在getView中执行耗时操作，不要在getView中直接加载图片，要异步执行，否则肯定会导致卡顿；

- 控制异步任务的执行频率：在列表滑动的时候停止加载图片，等列表停下来以后再加载图片。如：
```java
listView.setOnScrollListener(new AbsListView.OnScrollListener() {
    @Override
    public void onScrollStateChanged(AbsListView view, int scrollState) {
        if (scrollState == AbsListView.OnScrollListener.SCROLL_STATE_IDLE){
            isIdle = true;//是否静止的标志
            mAdapter.notifyDataSetChanged();
        }else {
            isIdle = false;
        }
    }

    @Override
    public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount) {

    }
});
```

在getView中只有静止时才能加载图片：

```java
if (isIdle){
    //loadImg
}
```	 
- 使用硬件加速来解决莫名的卡顿问题，给Activity添加配置android:hardwareAccelerated="true"。





