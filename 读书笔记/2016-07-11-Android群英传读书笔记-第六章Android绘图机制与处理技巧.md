---
layout:     post
title:      "Android群英传读书笔记-第六章Android绘图机制与处理技巧"
date:       2016-07-11 21:10:25
author:     "afayp"
catalog:    true
tags:
    - Android
    - 读书笔记
---


# 屏幕的尺寸信息
见之前文章。

<!--more-->

# 2D绘图基础
paint，canvas的使用，看这篇：[https://github.com/GcsSloop/AndroidNote](https://github.com/GcsSloop/AndroidNote)


# Android XML 绘图
XML在安卓系统中可不仅仅是JAVA中的一个布局文件配置列表，在安卓开发者的手头上他甚至可以变成一张画，一幅画，Android开发者给XML提供了几个强大的属性。

## Bitmap
直接这样引用图片就可以将图片直接转化成Bitmap让我们在程序中使用:
```java
<?xml version="1.0" encoding="utf-8"?>
<bitmap xmlns:android="http://schemas.android.com/apk/res/android"
    android:src="@mipmap/ic_launcher">
</bitmap>
```
## Shape
通过Shape可以绘制各种图形
```java
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"

    android:shape="rectangle">
    <!--默认是rectangle-->

    <!--当shape= rectangle的时候使用-->
    <corners
        android:bottomLeftRadius="1dp"
        android:bottomRightRadius="1dp"
        android:radius="1dp"
        android:topLeftRadius="1dp"
        android:topRightRadius="1dp" />
    <!--半径，会被后面的单个半径属性覆盖，默认是1dp-->

    <!--渐变-->
    <gradient
        android:angle="1dp"
        android:centerColor="@color/colorAccent"
        android:centerX="1dp"
        android:centerY="1dp"
        android:gradientRadius="1dp"
        android:startColor="@color/colorAccent"
        android:type="linear"
        android:useLevel="true" />

    <!--内间距-->
    <padding
        android:bottom="1dp"
        android:left="1dp"
        android:right="1dp"
        android:top="1dp" />

    <!--大小，主要用于imageview用于scaletype-->
    <size
        android:width="1dp"
        android:height="1dp" />

    <!--填充颜色-->
    <solid android:color="@color/colorAccent" />

    <!--指定边框-->
    <stroke
        android:width="1dp"
        android:color="@color/colorAccent" />
    <!--虚线宽度-->
    android:dashWidth= "1dp"

    <!--虚线间隔宽度-->
    android:dashGap= "1dp"

</shape>
```

## Layer
在Android中，可以使用Layer实现图层的效果，图层会依次叠加
```java
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <!--图片1-->
    <item android:drawable="@mipmap/ic_launcher"/>
 
    <!--图片2-->
    <item
        android:bottom="10dp"
        android:top="10dp"
        android:right="10dp"
        android:left="10dp"
        android:drawable="@mipmap/ic_launcher"
        />
 
</layer-list>
```

## Selector
Selector的作用是帮助开发者实现静态View的反馈，通过设置不同的属性呈现不同的效果
```java
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- 默认时候的背景-->
    <item android:drawable="@mipmap/ic_launcher" />

    <!-- 没有焦点时候的背景-->
    <item android:drawable="@mipmap/ic_launcher" android:state_window_focused="false" />

    <!-- 非触摸模式下获得焦点并点击时的背景图片-->
    <item android:drawable="@mipmap/ic_launcher" android:state_pressed="true" android:state_window_focused="true" />

    <!-- 触摸模式下获得焦点并点击时的背景图片-->
    <item android:drawable="@mipmap/ic_launcher" android:state_focused="false" android:state_pressed="true" />

    <!--选中时的图片背景-->
    <item android:drawable="@mipmap/ic_launcher" android:state_selected="true" />

    <!--获得焦点时的图片背景-->
    <item android:drawable="@mipmap/ic_launcher" android:state_focused="true" />

</selector>
```

# Android绘图技巧

## Canvas
> 
• Canvas.save()——将之前绘制的所有图像保存起来，让后续操作在一个新的图层上绘制，就像PS中的图层概念一样
• Canvas.restore()——将我们save()之后绘制的所有图像与save()之前的图像进行合并。
• Canvas.translate()——将坐标系平移到指定位置
• Canvas.roate()——旋转坐标系

## Layer图层
> 
通过saveLayer()方法，saveLayerAlpha()将一个图层入栈，使用restore(）方法，restoreToCount（）方法将一个图层出栈，入栈的时候，后面的所有才做都是发生在这个图层上的，而出栈的时候，则会把图层绘制在上层Canvas上。
![](http://oeiu2t0ur.bkt.clouddn.com/20160327215159839.png)

# Android图像处理之色彩特效处理
Android对于图片的处理，最常用的数据结构是位图——Bitmap，他包含了一张图片的所有数据，整个图片都是由点阵和颜色值去组成的，所谓的点阵就是一个包含像素的矩形，每一个元素对应着图片的一个像素，而颜色值——ARGB，分别对应透明度红，绿，蓝，这四个通用的分量，他们共同决定了每个像素点显示的颜色

![](http://oeiu2t0ur.bkt.clouddn.com/20160327221742455.png)


## 1.色彩矩阵分析
在色彩处理中，我们通常用三个角度描述一张图片
• 色调——物体传播的颜色
• 饱和度——颜色的纯度，从0-100来进行描述
• 亮度——颜色的相对，明暗程度
而在Android中，系统会使用一个颜色矩阵——ColorMatrix,来处理这些色彩的效果，Android中的颜色矩阵是4X5的数字矩阵，他用来对颜色色彩进行处理，而对于每一个像素点，都有一个颜色分量矩阵来保存ARGB值。
![](http://oeiu2t0ur.bkt.clouddn.com/20160327222645743.png)

图中矩阵A就是一个4X5的颜色矩阵，在Android中，他们会以一段一维数组的形式来存储[a,b,c,d,e,f,g,h,i,j,k,l,m,n,o,p,q,r,s,t],则C就是一个颜色矩形分量，在处理图像中，使用矩阵乘法运算AC处理颜色分量矩阵如下图
![](http://oeiu2t0ur.bkt.clouddn.com/20160327223427533.png)
对于颜色矩阵：
![](http://oeiu2t0ur.bkt.clouddn.com/20160327223951229.png)

• 第一行的abcde用来决定新的颜色值R——红色
• 第二行的fghij用来决定新的颜色值G——绿色
• 第三行的kimno用来决定新的颜色值B——蓝色
• 第四行的pqrst用来决定新的颜色值A——透明

当我们要变换颜色值的时候通常有两种方法，一个是直接改变颜色的offset，即修改颜色的分量。另一种方法直接改变RGBA值的系数来改变颜色分量的值。


### 改变色光属性
图像的**色调**,**饱和度**，**亮度**这三个属性在图像处理中使用的非常多，因此在颜色矩阵中也封装了一些API来快速调用这些参数，而不用每次都去计算矩阵的值。系统封装了一个类——`ColorMatrix`,也就是前面所说的颜色矩阵。通过这个类，可以很方便地通过改变矩阵值来处理颜色的效果，创建一个ColorMatrix对象非常简单代码如下。
`ColorMatrix colorMatrix = new ColorMatrix();`
下面我们来处理不同的色光属性

- 色调
安卓安卓系统提供了setRotate()帮助我们设置三个颜色的色调，第一个参数系统分别使用0,1,2来代表red green blue 三个颜色的处理；第二个参数就是需要处理的值了
```java
ColorMatrix hueMatrix = new ColorMatrix();
hueMatrix .setRotate(0, 2);
hueMatrix .setRotate(1, 4);
hueMatrix .setRotate(2, 3);
```
- 饱和度 
Android系统中提供了setSaturation（）来设置颜色的饱和度，参数即代表饱和度的值，当饱和度为0的时候变成灰度图像。
```java
ColorMatrix saturationMatrix = new ColorMatrix();
saturationMatrix .setSaturation(10);
```
- 亮度 
当三原色以相同的比例进行混合的时候，就会显示出白色，系统也正在使用这个原理来改变一个图像的亮度,代码如下，当亮度为零时图像会变成全黑。
```java
ColorMatrix lumMatrix = new ColorMatrix();
lumMatrix .setScale(10, 10, 10, 10);
```
除了单独使用上面的三种方法来经颜色效果的处理之外,Android系统还封装了矩阵的乘法运算，它提供了PostConcat方法来将矩阵的作用效果混合,从而叠加处理效果，代码如下。
```java
ColorMatrix colorMatrix = new ColorMatrix();
colorMatrix.postConcat(hueMatrix);
colorMatrix.postConcat(saturationMatrix);
colorMatrix.postConcat(lumMatrix);
```
下面看一个综合设置的例子：
```java
public void handleImgEffect(Bitmap bm, float hue, float saturation, float lum) {
    //bm代表原图，Android系统不允许直接修改原图，必须通过原图创建一个同样大小的Bitmap并将原图绘制到该Bitmap上。
    Bitmap bmp = Bitmap.createBitmap(bm.getWidth(), bm.getHeight(),     Bitmap.Config.ARGB_8888);//创建一个相同大小的Bitmap
    Canvas canvas = new Canvas(bmp);//将bmp铺到canvas上
    Paint paint = new Paint();
    //色调
    ColorMatrix hueMatrix = new ColorMatrix();
    hueMatrix.setRotate(0, hue);
    hueMatrix.setRotate(1, hue);
    hueMatrix.setRotate(2, hue);
    //饱和度
    ColorMatrix saturationMatrix = new ColorMatrix();
    saturationMatrix.setSaturation(saturation);
    //亮度
    ColorMatrix lumMatrix = new ColorMatrix();
    lumMatrix .setScale(lum, lum, lum, 1);
    //汇总
    ColorMatrix imgMatrix = new ColorMatrix();
    imgMatrix.postConcat(hueMatrix);
    imgMatrix.postConcat(saturationMatrix);
    imgMatrix.postConcat(lumMatrix);
    //通过Paint类的setColorFilter将配置好的效果设置进去
    paint.setColorFilter(new ColorMatrixColorFilter(imgMatrix));
    canvas.drawBitmap(bm,0,0,paint);
}
```

### 2.Android颜色矩阵——ColorMatrix
调整颜色矩阵可以改变一张图片的颜色属性，图像处理很大程度上就是寻找处理图像的颜色矩阵，不仅仅可以通过Android系统提供的API完成，我们自己也可以修改。

### 3.常用图象颜色矩阵处理效果
> 
• 灰度效果
• 反转效果
• 怀旧效果
• 去色效果
• 高饱和度

具体看书P132

### 4.像素点分析
作为更加精确的图像处理方式，可以通过改变每个像素点的具体ARGB值，达到处理一张图片效果的目的，这里要注意的是，传递进来的原始图片是不能修改的，一般根据原始图片生成一张新的图片来修改，在Android中，系统系统提供了`Bitmap.getPixels()`方法来帮我们提取整个Bitmap中的像素密度点，并保存在一个数组中，该方法如下：
`bitmap.getPixels(pixels , offset,stride, x, y, width, height);`
每个参数的含义：
• pixels ——接收位图颜色值的数组，
• offset——写入到pixels[]第一个像素索引值，
• stride——pixels[]中的行间距
• x——从位图中读取的第一个像素的ｘ坐标
• y——从图中读取的第一个像素的的y坐标
• width——从每一行读取的像素宽度
• height——读取的行数
一般情况下，我们拿到bitmap，提取像素点的代码这么写（width，height表示图片宽高）
```java
Int[] oldPx = new int[width,height];
bitmap.getPixels(oldPx, 0, width, 0, 0, width, height);
```
拿到像素点后，我们就可以获取每一个像素具体的ARGB值，代码如下：
```java
for (int i = 0; i < width * height; i++) {
        int color = oldPx[i];
        r = Color.red(color);
        g = Color.green(color);
        b = Color.blue(color);
        a = Color.alpha(color);
	//可以根据需要改变argb的值，组成一个新的newPx
	newPx[i] = Color.argb(a,r,g,b);
}
```
最后将处理后的像素点数组重新set给我们的bitmap，从而完成图片的处理。
`bmp.setPixels(newPx, 0, width, 0, 0, width, height);`

### 5.常用图像图片像素点处理效果
- 底片效果
算法（B表示一个像素点）：
```java
B.r = 255 - B.r;
B.g = 255 - B.g;
B.b = 255 - B.b;
```
具体代码：
```java
public  Bitmap handleImageNegative(Bitmap bm) {
    int width = bm.getWidth();
    int height = bm.getHeight();
    int color;
    int r, g, b, a;
 
    Bitmap bmp = Bitmap.createBitmap(width, height, Bitmap.Config.ARGB_8888);
 
    int[] oldPx = new int[width * height];
    int[] newPx = new int[width * height];
 
    bm.getPixels(oldPx, 0, width, 0, 0, width, height);
 
    for (int i = 0; i < width * height; i++) {
        color = oldPx[i];
        r = Color.red(color);
        g = Color.green(color);
        b = Color.blue(color);
        a = Color.alpha(color);
 
        r = 255 - r;
        g = 255 - g;
        b = 255 - b;
 
        if (r > 255) {
            r = 255;
        } else if (r < 0) {
            r = 0;
        }
 
        if (g > 255) {
            g = 255;
        } else if (g < 0) {
            g = 0;
        }
 
        if (b > 255) {
            b = 255;
        } else if (b < 0) {
            b = 0;
        }
        newPx[i] = Color.argb(a, r, g, b);
    }
    bmp.setPixels(newPx, 0, width, 0, 0, width, height);
    return bmp;
}
```
- 老照片效果
算法：
```java
r = (int) (0.393 * r + 0.769 * g + 0.189 * b);
g = (int) (0.349 * r + 0.686 * g + 0.168 * b);
b = (int) (0.272 * r + 0.534 * g + 0.131 * b);
```
- 浮雕效果
算法：
```java
B.r = C.r - B.r + 127;
B.g = C.g - B.g + 127;
B.b = C.b - B.b + 127;
```

# Android图像处理之图像特效处理
## 1.Android变形矩阵——Matrix
对于图像的处理，Android系统提供了ColorMatrix颜色矩阵来帮助我们进行图像处理，而对于图像的图形变换，Android系统也可以通过矩阵来帮忙，每个像素点都表达了其xy的信息，Android的变化就是一个3X3的矩阵。
见 Matrix。。。
## 2.像素块分析
与前面色彩的处理类似，使用了颜色矩阵和像素点处理两种方式来进行图像处理，图像特效方面的处理方式也有两种，即前面讲的使用变形矩阵来进行图像变换和马上要使用到的drawBitmapMesh()方法来处理。drawBitmapMesh处理与前面操纵像素点改变色彩的原理类似，只是把图像分成了一个个的小块，然后改变每个小块来修改图像。该方法如下：
```java
canvas.drawBitmapMesh(Bitmap bitmap,int meshWidth,int meshHeight,float [] verts,int vertOffset,int [] color,int colorOffset,Paint paint);
```
# Android图像处理之画笔特效处理
画笔paint的高级属性

## 1. PorterDuffXfermode
先来看Xfermode，它表示一种图像转换模式，有三个子类：
	- AvoidXfermode  指定了一个颜色和容差，强制Paint避免在它上面绘图(或者只在它上面绘图)。
	- PixelXorXfermode  当覆盖已有的颜色时，应用一个简单的像素异或操作。
	- PorterDuffXfermode  这是一个非常强大的转换模式，使用它，可以使用图像合成的16条Porter-Duff规则的任意一条来控制Paint如何与已有的Canvas图像进行交互。
要应用转换模式，可以使用`setXferMode`方法，如下所示：
```java
AvoidXfermode avoid = new AvoidXfermode(Color.BLUE, 10, AvoidXfermode.Mode. AVOID);
mPaint.setXfermode(avoid);  
```

PorterDuffXfermode有16中枚举类,见下图：
![](http://oeiu2t0ur.bkt.clouddn.com/%E6%9C%AA%E5%91%BD%E5%90%8D%E5%9B%BE%E7%89%87.png)

解释：
> 
	1.PorterDuff.Mode.CLEAR  
	  所绘制不会提交到画布上。
	2.PorterDuff.Mode.SRC
	   显示上层绘制图片
	3.PorterDuff.Mode.DST
	  显示下层绘制图片
	4.PorterDuff.Mode.SRC_OVER
	  正常绘制显示，上下层绘制叠盖。
	5.PorterDuff.Mode.DST_OVER
	  上下层都显示。下层居上显示。
	6.PorterDuff.Mode.SRC_IN
	   取两层绘制交集。显示上层。
	7.PorterDuff.Mode.DST_IN
	  取两层绘制交集。显示下层。
	8.PorterDuff.Mode.SRC_OUT
	 取上层绘制非交集部分。
	9.PorterDuff.Mode.DST_OUT
	 取下层绘制非交集部分。
	10.PorterDuff.Mode.SRC_ATOP
	 取下层非交集部分与上层交集部分
	11.PorterDuff.Mode.DST_ATOP
	 取上层非交集部分与下层交集部分
	12.PorterDuff.Mode.XOR
	  异或：去除两图层交集部分
	13.PorterDuff.Mode.DARKEN
	  取两图层全部区域，交集部分颜色加深
	14.PorterDuff.Mode.LIGHTEN
	  取两图层全部，点亮交集部分颜色
	15.PorterDuff.Mode.MULTIPLY
	  取两图层交集部分叠加后颜色
	16.PorterDuff.Mode.SCREEN
	  取两图层全部区域，交集部分变为透明色

当然，这些模式也不是经常的用到，用到最多的是，使用一张图片作为另一张图片的遮罩，通过控制遮罩层的图形，来控制下面被遮罩的显示效果，其中最常用的就是通过`DST_IN.SRC_IN`模式来实现将一个矩形变成圆角图片的效果
```java
mBitmap = BitmapFactory.decodeResource(getResources(), R.drawable.nice);//原图
mOut = Bitmap.createBitmap(mBitmap.getWidth(), mBitmap.getHeight(), Bitmap.Config.ARGB_8888);//遮罩
Canvas canvas = new Canvas(mOut);
mPaint = new Paint();
mPaint.setAntiAlias(true);
canvas.drawRoundRect(0, 0, mBitmap.getWidth(), mBitmap.getHeight(), 80, 80, mPaint);
mPaint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_IN));//取交集，显示上层的原图
canvas.drawBitmap(mBitmap,0,0,mPaint);
```
实现刮刮卡效果。。。

## 2.Shader
Shader又被称为着色器。渲染器，它可以实现渲染，渐变等效果，Android中的Shader包括以下几种
> 
• BitmapShader:位图Shader
• LinearGradient:线性Shader
• RadialGradient:光束Shader
• SweepGradient:梯形Shader
• ComposeShader:混合Shader

先来看`BitmapShader`
除了第一个之外，其他的都是比较正常的，实现了名副其实的渐变，渲染效果，而与其他Shader的效果不同的是，BitmapShader所产生的是一个图像，有点类似PS里面的图像填充，他的作用是通过Paint对画布进行制定bitmap的填充，填充式有一下几个模式供选择。

	• CLAMP拉伸——拉伸的是图片最后的哪一个像素，不断重复
	• REPEAT重复——横向，纵向不断重复
	• MIRROR镜像——横向不断翻转重复，纵向不断翻转重复
	
这里最常用的是CLAMP拉伸模式，虽然他会拉伸最后一个像素，但是只要将图片设置成一个固定的大小，就可以避免这种拉伸，下面我们来举个例子：
```java
mBitmap = BitmapFactory.decodeResource(getResources(),R.drawable.nice);
mBitmapShader = new BitmapShader(mBitmap, Shader.TileMode.CLAMP,Shader.TileMode.CLAMP);
mPaint = new Paint();
mPaint.setShader(mBitmapShader);
canvas.drawCircle(500,250,200,mPaint);
```
上面这段代码的意思是用一张图片创建了一支具有图像填充功能的画笔，并用这只画笔绘制了一个圆形。

LinearGradient——线性渐变。
```java
mPaint = new Paint();
mPaint.setShader(new LinearGradient(0,0,400,400, Color.BLUE,Color.YELLOW, Shader.TileMode.REPEAT));
canvas.drawRect(0,0,400,400,mPaint);
```
上面代码画了一个从（0,0）到（400,400）的蓝色到黄色的渐变效果
其他几种用法基本相同，就不讲了。

通常这些渐变效果不会直接使用在程序里，而是把这种渐变效果作为一个遮罩层来使用，结合前面的`PorterDuffXfermode`。下面演示一个用`LinearGradient和PorterDuffXfermode来创建一个具有倒影效果的图片。
```java
public class ReflectView extends View {

    private Bitmap mSrcBitmap;
    private Paint mPaint;
    private PorterDuffXfermode mXfermode;
    private Bitmap mRefBitmap;

    public ReflectView(Context context, AttributeSet attrs) {
        super(context, attrs);
        initRes();
    }

    private void initRes() {
        mSrcBitmap = BitmapFactory.decodeResource(getResources(), R.mipmap.ic_launcher);
        Matrix matrix = new Matrix();
        matrix.setScale(1F,-1F);//图片垂直翻转（倒影效果），镜面效果的话.setScale(-1F,1F)
        mRefBitmap = Bitmap.createBitmap(mSrcBitmap, 0, 0, mSrcBitmap.getWidth(), mSrcBitmap.getHeight(), matrix, true);
        mPaint = new Paint();
        mPaint.setShader(new LinearGradient(0,mSrcBitmap.getHeight(),0,mSrcBitmap.getHeight()+mSrcBitmap.getHeight()/4
                ,0XDD000000, 0X10000000, Shader.TileMode.CLAMP));
        mXfermode = new PorterDuffXfermode(PorterDuff.Mode.DST_IN);//取交集显示下层

    }

    @Override
    protected void onDraw(Canvas canvas) {
        canvas.drawColor(Color.BLACK);
        canvas.drawBitmap(mSrcBitmap,0,0,null);
        canvas.drawBitmap(mRefBitmap,0,mSrcBitmap.getHeight(),null);
        mPaint.setXfermode(mXfermode);
        canvas.drawRect(0,mSrcBitmap.getHeight(),mRefBitmap.getWidth(),mSrcBitmap.getHeight()*2,mPaint);
        mPaint.setXfermode(null);

    }
}
```

## 3.PathEffect
要想了解PathEffect，还是直接看一张图比较直观吧
![](http://oeiu2t0ur.bkt.clouddn.com/20160402205535417.png)

PathEffect就是指，用各种笔触效果绘制路径，Android系统提供的从上到下依次是：
> 
	• 没效果
	• CornerPathEffect:拐弯处变圆滑，具体圆滑程度由参数决定
	• DiscretePathEffect： 线段上有杂点
	• DashPathEffect：虚线，用一个数组来设置各个点之间的间隔，重复这样的间隔绘制，另一个参数phase用来控制绘制时数组的偏移量，用来实现路径的动态效果
	• PathDashPathEffect：与DashPathEffect类似，且可以设置显示虚线点的图形
	• ComposePathEffect：将前面的任意两种路径特性组合起来

一个简单例子： 
```java
mEffect[0] = null;
mEffect[1] = new CornerPathEffect(30);
mEffect[2] = new DiscretePathEffect(3.0F,5.0F);
mEffect[3] = new DashPathEffect(new float[]{20,10,5,10},0);
Path path = new Path();
path.addRect(0,0,8,8,Path.Direction.CCW);
mEffect[4]= new PathDashPathEffect(path,12,0,PathDashPathEffect.Style.ROTATE);
mEffect[5] = new ComposePathEffect(mEffect[3],mEffect[1]);
for (int i = 0; i<mEffect.length;i++){
    mPaint.setPathEffect(mEffect[i]);
    canvas.drawPath(mPath,mPaint);
    canvas.translate(0,200);
}
```

# View之孪生兄弟——SurfaceView
##　1.SurfaceView和View的区别
Android系统提供了VieW进行绘图处理, vieW可以满足大部分的绘图需求，但在某些时却也有些心有余而力不足,特别是在进行一些开发的时候。 我们知道,View通过刷新来重绘视图, Android系统通过发出VSYNC信号来进行屏幕的重绘, 刷新的间隔时间为I6ms。在16ms内View完成了你所需要执行的所有操作,那么用户在视觉上, 就不会产生卡顿的感觉:而如果执行的操作逻辑太多,特别是需要频繁刷新的界面上，例如游戏界面,那么就塞主线程，从而导致画面卡顿，很多时候,在自定义View的Log中经常会看见如下示的警告
```java
Skipped 47 frames! The application may be doing too much work on its main thread
```
这些警告的产生，很多情况下就是因为在绘制过程中, 处理逻辑太多造成的为了避免这一问题的产生一 Android系统提供了surfacevicW组件来解决这个问题,可以说是View的孪生兄弟,但它与View还是有所不同的,它们的**区别**主要体现

	• 1.View主要用于自动更新的情况下,而surfaceVicw主要适用于被动更新,例如频繁刷新
	• 2.View在主线程中刷新，而surfaceView通常会通过一 个子线程来进行页面刷新。
	• 3.View在绘制的时候没有双缓冲机制,而surfaceVicw在底层实现机制中就已经实现了双缓冲机制；
	
总结起来，如果你的自定义View需要频繁刷新,或者刷新时数据处理量比较大，那么你就可以考虑使用surfaceVicw取代View了

## 2.surfaceView的使用
通常情况下 使用以下步骤来创建一个`surfaceView`的模板。

1.创建surfaceView 
    创建自定义的surfaceView，继承SurfaceView，并实现它的两个接口
```java
    public class SurfaView extends SurfaceView implements SurfaceHolder.Callback,Runnable
```
    实现三个回调方法:
```java
    @Override
    public void surfaceCreated(SurfaceHolder holder) {
    }
    @Override
    public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
    }
    @Override
    public void surfaceDestroyed(SurfaceHolder holder) {
    }
```
    分别对应的是SurfaceView的创建，改变和销毁的过程
    对于Runnable接口，会实现他的回调
```
    @Override
    public void run() {
    }
```
2.初始化SurfaceView
通常需要三个成员变量：
```java
//SurfaceHolder
private SurfaceHolder mHolder;
//用于绘制的Canvas
private Canvas mCanvas;
 //子线程标志位
private boolean mIsDrawing;
```
对于SurfaceHolder 这么初始化：
```java
mHolder = getHolder();
mHolder.addCallback(this);
```
canvas用来画图，mIsDrawing用来控制子线程

3.使用SurfaceView
> 
通过SurfaceHolder对象的lockCanvas方法，就可以获得当前Canvas绘图对象，接下来，就可以与在View中进行的绘制操作一样进行绘制了，不过这里要注意的是Canvas对象还是继续上次的Canvas对象,而不是一 个新的对象。因此，之前的操作都将被保留，如果需要擦除,则可以在绘制前, 通过drawColoro方法来进行清屏操作。
> 
绘制的时候，充分利用SurfaceView的回调方法，在`surfaceCreated`方法中开启子线程进行绘制，而子线程开启了一个`while(mIsDrawing)`的循环来不停的绘制，而在具体的逻辑中，通过`lookCanvas()`方法获取Canvas对象进行绘制，并通过`unlockCanvasAndPost（mCanvas）`方法对画布的内容进行提交，整个代码模块如下：
```java
public class TemplateSurfaceView extends SurfaceView implements SurfaceHolder.Callback,Runnable {
    private SurfaceHolder mHolder;
    private Canvas mCanvas;
    private boolean mIsDrawing;


    public TemplateSurfaceView(Context context) {
        super(context);
        initView();
    }

    public TemplateSurfaceView(Context context, AttributeSet attrs) {
        super(context, attrs);
        initView();
    }

    private void initView() {
        mHolder = getHolder();
        mHolder.addCallback(this);
        setFocusable(true);
        setFocusableInTouchMode(true);
        setKeepScreenOn(true);
    }

    @Override
    public void surfaceCreated(SurfaceHolder surfaceHolder) {
        mIsDrawing = true;
        new Thread(this).start();
    }

    @Override
    public void surfaceChanged(SurfaceHolder surfaceHolder, int i, int i1, int i2) {

    }

    @Override
    public void surfaceDestroyed(SurfaceHolder surfaceHolder) {
        mIsDrawing = false;

    }

    @Override
    public void run() {
        while (mIsDrawing){
            draw();
        }

    }

    private void draw() {
        try {
            mCanvas = mHolder.lockCanvas();
            //draw something
        }catch (Exception e){

        }finally {
            if (mCanvas != null)
                mHolder.unlockCanvasAndPost(mCanvas);//放到finally中保证每次都将内容提交
        }

    }
}
```
一个用surfaceView实现的画图板：
```java
public class SketchpadSurfaceView extends SurfaceView implements SurfaceHolder.Callback,Runnable{
    //SurfaceHolder
    private SurfaceHolder mHolder;
    //用于绘制的Canvas
    private Canvas mCanvas;
    //子线程标志位
    private boolean mIsDrawing;
    private int x, y = 0;
    private Path mPath;
    private Paint mPaint;

    public SketchpadSurfaceView(Context context) {
        super(context);
        initView();
    }

    public SketchpadSurfaceView(Context context, AttributeSet attrs) {
        super(context, attrs);
        initView();
    }

    public SketchpadSurfaceView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initView();
    }

    private void initView() {
        mPath = new Path();
        mPaint = new Paint();
        mPaint.setStrokeWidth(10);
        mPaint.setAntiAlias(true);
        mPaint.setStyle(Paint.Style.STROKE);
        mHolder = getHolder();
        mHolder.addCallback(this);
        setFocusable(true);
        setFocusableInTouchMode(true);
        setKeepScreenOn(true);
    }

    @Override
    public void surfaceCreated(SurfaceHolder surfaceHolder) {
        mIsDrawing = true;
        new Thread(this).start();
    }

    @Override
    public void surfaceChanged(SurfaceHolder surfaceHolder, int i, int i1, int i2) {

    }

    @Override
    public void surfaceDestroyed(SurfaceHolder surfaceHolder) {
        mIsDrawing = false;

    }

    @Override
    public void run() {
        long start = System.currentTimeMillis();
        while (mIsDrawing){
            draw();
        }
        long end = System.currentTimeMillis();
        //有时候绘制的不用这么频繁，一般100ms绘制一次即可，如果draw方法里面的耗时小于100ms，就sleep一下
        if (end - start < 100){
            try {
                Thread.sleep(100-(end-start));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

    }

    private void draw() {
        try {
            mCanvas = mHolder.lockCanvas();
            mCanvas.drawColor(Color.WHITE);
            mCanvas.drawPath(mPath, mPaint);
        } catch (Exception e) {

        } finally {
            if (mCanvas != null) {
                //提交
                mHolder.unlockCanvasAndPost(mCanvas);
            }
        }
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int x = (int) event.getX();
        int y = (int) event.getY();
        switch (event.getAction()){
            case MotionEvent.ACTION_DOWN:
                mPath.moveTo(x,y);
                break;
            case MotionEvent.ACTION_MOVE:
                mPath.lineTo(x,y);
                break;
        }
        return true;
    }
}
```
