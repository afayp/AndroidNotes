---
layout:     post
title:      "Android群英传读书笔记-第七章Android动画机制和使用技巧"
date:       2016-08-09 22:10:25
author:     "afayp"
catalog:    true
tags:
    - Android
    - 读书笔记
---



前面的view动画，属性动画，布局动画，插值器参考动画章节，不赘述。

<!--more-->

# 自定义动画
基本使用参考这里前面的动画章节，重点讲一下3D效果Camera的使用。

使用Canmera来实现一个3D的效果，要注意的是，这里所指的Camera不是相机，而是android.graphics.Camera这个类，他封装了openGL的3D动画,从而可以非常方便的创建3D效果。把Camera想象成真实的摄像机，当物体固定在某处时，只要移动摄像机就能拍摄到具有立体感的图像，因此通过它就能实现各种3D效果。

栗子：
```java
public class CameraAnimation extends Animation {
    private int mCenterWidth, mCenterHeight;
    private Camera mCamera;
    private float mRotateY;

    @Override
    public void initialize(int width, int height, int parentWidth, int parentHeight) {
        super.initialize(width, height, parentWidth, parentHeight);
        setDuration(4000);
        setFillAfter(true);
        setInterpolator(new BounceInterpolator());
        mCenterWidth = mCenterWidth/2;
        mCenterHeight = mCenterHeight/2;
        mCamera = new Camera();
        mRotateY = 45f;

    }

    @Override
    protected void applyTransformation(float interpolatedTime, Transformation t) {
        super.applyTransformation(interpolatedTime, t);
        Matrix matrix = t.getMatrix();
        mCamera.save();
        mCamera.rotateY(mRotateY * interpolatedTime);
        mCamera.getMatrix(matrix);
        mCamera.restore();
        matrix.preTranslate(mCenterWidth,mCenterHeight);
        matrix.postTranslate(-mCenterWidth,-mCenterHeight);
    }
}
```
Camera更多用法参见：
[初探android的Camera和Matrix](http://blog.csdn.net/imyfriend/article/details/8045973)
[android.graphics.Camera实现图像的旋转、缩放，配合Matrix实现图像的倾斜](http://blog.csdn.net/sodino/article/details/6823315)
[Android APIDemos 研读之二：android.graphics.Camera](http://blog.csdn.net/sharetop/article/details/5277655)


# Android 5.X SVG矢量动画机制
> 
Google在Android5.X中增加了对SVG矢量图形的支持，添加了对< path >标签的支持，从而让开发者可以使用SVG来创建更加丰富的动画效果，那么SVG对传统的Bitmap，究竟有什么好处呢？bitmap通过每个像素点上存储色彩信息来表达图像，而SVG是一个绘图标准，与之相对，最大的优势是SVG放大不会失真，而且bitmap需要不同分辨率适配，SVG不需要。

1.< path >标签
使用< path >标签来创建SVG，就是用指令的方式来控制一支画笔，列入，移动画笔来到某一个坐标位置，画一条线，画一条曲线，结束，< path >标签所支持的指令大致有一下几种:
> 
M = moveto(M X,Y):将画笔移动到指定的坐标位置，但未发生绘制
L = lineto(L X,Y):画直线到指定的位置
H = horizontal lineto( H X):画水平线到指定的X坐标位置
V = vertical lineto(V Y ):画垂直线到指定的Y坐标
C = curveto(C ,X1,Y1,X2,Y2,ENDX,ENDY):三次贝塞尔曲线
S = smooth curveto(S X2,Y2,ENDX,ENDY):三次贝塞尔曲线
Q = quadratic Belzier curve(Q X Y,ENDX,ENDY):二次贝塞尔曲线
T = smooth quadratic Belzier curvrto(T,ENDX,ENDY):映射前面路径的重点
A = elliptical Are(A RX,RY,XROTATION,FLAG1FLAG2,X,Y):弧线
Z = closepath() 关闭路径

使用上面的指令时，需要注意的几点
> 
坐标轴以（0,0）位中心，X轴水平向右，Y轴水平向下
所有指令大小写均可，大写绝对定位，参照全局坐标系，小写相对定位，参照父容器坐标系
指令和数据间的空格可以无视
同一指令出现多次可以用一个

2.SVG常见指令
L
> 
绘制直线的指令是“L”，代表从当前点绘制直线到给定点，“L”之后的参数是一个点坐标，如“L 200 400”绘制直线，同时，还可以使用“H”和“V”指令来绘制水平竖直线，后面的参数是x坐标个y坐标

M
> 
M指令类似Android绘图中的path类moveto方法，即代表画笔移动到某一点，但并不发生绘图动作

A
> 
A指令是用来绘制一条弧线，且允许弧线不闭合，可以把A指令绘制的弧度想象成椭圆的某一段A指令一下有七个指令
RX,RY指所有的椭圆的半轴大小
XROTATION 指椭圆的X轴和水平方向顺时针方向的夹角，可以想象成一个水平的椭圆饶中心点顺时针旋转XROTATION 的角度
FLAG1 只有两个值，1表示大角度弧度，0为小角度弧度
FLAG2 只有两个值，确定从起点到终点的方向1顺时针，0逆时针
X,Y为终点坐标

SVG的指令参数非常的复杂,一般都用编辑器，不会自己写。

3.SVG编辑器
	在线： [http://editor.method.ac/](http://editor.method.ac/)
	离线：Inkscape
	
4.Android中使用SVG

Google在Android5.X后给我们提供了两个新的API来支持SVG
	• VectorDrawable
	• AnimatedVectorDrawable
其中，VectorDrawable可以让你创建基于XML的SVG图像，并且结合AnimatedVectorDrawable来实现动画效果

- VectorDrawable
在XML中创建一个静态的SVG，通过这个结构
![](http://oeiu2t0ur.bkt.clouddn.com/20160416231719716.png)

具体代码：
```java
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="200dp"
    android:height="200dp"
    android:viewportHeight="100"
    android:viewportWidth="100">
 
    <group
        android:name="svg1"
        android:rotation="0">
        <path
            android:fillColor="@android:color/holo_blue_light"
            android:pathData="M 25 50 a 25,25 0 1,0 50,0" />
 
    </group>
 
</vector>
```
解释：
> 
	width,和height是表示SVG图像的具体大小，后面的是表示SVG图像划分的比例，后面再绘制path时所使用的参数，就是根据这两个值来进行转换的，比如上面的代码，将200dp划分100份，如果在绘图中使用坐标（50,50），则意味着该坐标为正中间。所以这两组宽高的比例要一致，不然图片就会压缩，形变。postData就是绘制SVG图形所用到的指令。fillColor表示填充图形，非填充可以用strokeColor和strokeWidth属性。最后只要把这个xml文件作为src给imageView就可以显示出来了。

- AnimatedVectorDrawable

> 
AnimatedVectorDrawable的作用是给VectorDrawable提供动画效果，Google的工程师将AnimatedVectorDrawable比喻一个胶水，通过AnimatedVectorDrawable来连接静态的VectorDrawable和动态的objectAnimator。

具体用法见书上。

# android动画特效
介绍了几个动画效果，都挺简单的，就不写了。