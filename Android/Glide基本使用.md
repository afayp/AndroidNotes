---
title: Glide基本使用
date: 2016-05-12 18:23:45
categories: Android
tags: Android
---


# Glide基本使用

完整教程见[github](https://github.com/bumptech/glide),本文只做一些记录。

### 加载方式

- url地址加载
`Glide.with(context).load("http://img2.3lian.com/2014/f6/173/d/51.jpg").into(imageView);`

- 资源文件加载
`Glide.with(context).load(R.mipmap.image).into(imageView);`

<!--more-->

- 本地文件加载
```
File file = new File(Environment.getExternalStorageDirectory() + File.separator +  "image", "image.jpg");
Glide.with(this).load(file).into(imageView);
```

- Uri加载
```
File file = new File(Environment.getExternalStorageDirectory() + File.separator +  "image", "image.jpg");
Uri uri = Uri.fromFile(file);
Glide.with(this).load(uri).into(imageView);//uri加载方式
```

### placeholder
`Glide.with(context).load("http://img2.3lian.com/2014/f6/173/d/55.jpg").placeholder(R.mipmap.place).into(imageView);`

### error
`Glide.with(context).load("http://img2.3lian.com/2014/f6/173/d/55.jpg").error(R.mipmap.error).into(imageView);`

### RequestListener加载监听
```
//设置错误监听
RequestListener<String,GlideDrawable> errorListener=new RequestListener<String, GlideDrawable>() {
    @Override
    public boolean onException(Exception e, String model, Target<GlideDrawable> target, boolean isFirstResource) {

        Log.e("onException",e.toString()+"  model:"+model+" isFirstResource: "+isFirstResource);
        imageView.setImageResource(R.mipmap.ic_launcher);
        return false;
    }

    @Override
    public boolean onResourceReady(GlideDrawable resource, String model, Target<GlideDrawable> target, boolean isFromMemoryCache, boolean isFirstResource) {
        Log.e("onResourceReady","isFromMemoryCache:"+isFromMemoryCache+"  model:"+model+" isFirstResource: "+isFirstResource);
        return false;
    }
} ;
```

### crossFade
`Glide.with(context).load("http://img2.3lian.com/2014/f6/173/d/51.jpg").error(R.mipmap.error).placeholder(R.mipmap.place).crossFade(2000).into(imageView);`
取消加载动画用`dontAnimate()`

### 图片大小调整
Glide加载图片大小是自动调整的，他根据ImageView的尺寸自动调整加载的图片大小，并且缓存的时候也是按图片大小缓存，每种尺寸都会保留一份缓存，如果图片不会自动适配到 ImageView，调用 `override(horizontalSize, verticalSize)`(px单位) 。这将在图片显示到 ImageView之前重新改变图片大小
`Glide.with(context).load("http://img2.3lian.com/2014/f6/173/d/51.jpg").dontAnimate().override(400,600).fitCenter().into(imageView);`

### 缩放
- centerCrop   
裁剪图片，当图片比ImageView大的时候，他把把超过ImageView的部分裁剪掉，尽可能的让ImageView 完全填充，但图像可能不会全部显示

- fitCenter   
它会自适应ImageView的大小，并且会完整的显示图片在ImageView中，但是ImageView可能不会完全填充

### Gif
加载Gif动画也是Glide的一大优势，它很简单的就能实现Gif的加载与显示
`Glide.with(context).load("http://img1.3lian.com/2015/w4/17/d/64.gif").into(imageView);`
显示Gif的第一帧使用asBitmap()方法;

### Glide网络加载方式
Glide内部默认是通过HttpURLConnection网络方式加载图片的，并且支持OkHttp,Volley。

- 集成OkHttp
```
//自动集成okhttp
compile 'com.github.bumptech.glide:okhttp-integration:1.4.0@aar'
compile 'com.squareup.okhttp:okhttp:2.2.0'
```
Gradle 会自动合并必要的 GlideModule 到Android.Manifest。Glide 会认可在 manifest 中的存在，然后使用 所集成的网络连接。

### 自定义动画

Glide默认提供了一个渐入渐出的动画效果，我们也可以自定义动画。Glide给我们提供了animate（）方法，我们可以通过此方法实现我们自定义的动画效果。
```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:fillAfter="false"
    android:duration="3000">

    <scale
        android:duration="@android:integer/config_longAnimTime"
        android:fromXScale="0.1"
        android:fromYScale="0.1"
        android:pivotX="50%"
        android:pivotY="50%"
        android:toXScale="1"
        android:toYScale="1"/>
    <rotate
        android:fromDegrees="0"
        android:toDegrees="90"
        android:pivotX="50%"
        android:pivotY="50%"
        />
</set>
```

使用：
`Glide.with(context).load("http://img2.3lian.com/2014/f6/173/d/51.jpg").animate(R.anim.anim).into(imageView);`

也可以通过Java代码实现动画，如下
```
ViewPropertyAnimation.Animator animator=new ViewPropertyAnimation.Animator() {
    @Override
    public void animate(View view) {
      view.setAlpha(0f);
        ObjectAnimator fadeAnim = ObjectAnimator.ofFloat( view, "alpha", 0f, 1f );
        fadeAnim.setDuration( 2500 );
        fadeAnim.start();
    }
};
Glide.with(context).load("http://img2.3lian.com/2014/f6/173/d/51.jpg").animate(animator).into(imageView);
```

### Target
Glide不但可以把图片、视频剧照、GIF动画加载到View，还可以加载到自定义的Target实现中。Target就是使用Glide获取到资源之后资源作用的目标，我们通常是用Glide加载完资源后显示到ImageView中，这个ImageView就是目标.

#### SimpleTarget
```
//SimpleTarget
SimpleTarget target = new SimpleTarget<Drawable>(){
    @Override
    public void onResourceReady(Drawable resource, GlideAnimation<? super Drawable> glideAnimation) {
        textView.setBackground(resource);
    }
};
Glide.with(context)
        .load("http://img2.3lian.com/2014/f6/173/d/51.jpg")
        .animate(animator)
        .into(target);
```
上面的代码我们将TextView作为Target,将加载的图片设为背景，对于SimpleTarget是接收的泛型数据，如果我们需要Bitmap对象，我们将泛型为Bitmap。  
我们还可以指定加载的宽和高，如下，设置宽和高都是100，单位是px
```
SimpleTarget target = new SimpleTarget<Drawable>(100,100){
    @Override
    public void onResourceReady(Drawable resource, GlideAnimation<? super Drawable> glideAnimation) {
        textView.setBackground(resource);
    }
};
```

#### ViewTarget
假设你有一个 Custom View。Glide 并不支持加载图片到自定义 view 中，因为并没有方法知道图片应该在哪里被设置。这时可以用 ViewTarget 实现。  

假设这么一个自定义view：
```
public class FutureStudioView extends FrameLayout {  
    ImageView iv;
    TextView tv;

    public void initialize(Context context) {
        inflate( context, R.layout.custom_view_futurestudio, this );

        iv = (ImageView) findViewById( R.id.custom_view_image );
        tv = (TextView) findViewById( R.id.custom_view_text );
    }

    public FutureStudioView(Context context, AttributeSet attrs) {
        super( context, attrs );
        initialize( context );
    }

    public FutureStudioView(Context context, AttributeSet attrs, int defStyleAttr) {
        super( context, attrs, defStyleAttr );
        initialize( context );
    }

    public void setImage(Drawable drawable) {
        iv = (ImageView) findViewById( R.id.custom_view_image );
        iv.setImageDrawable( drawable );
    }
}
```
加载：
```
private void loadImageViewTarget() {  
    FutureStudioView customView = (FutureStudioView) findViewById( R.id.custom_view );

    viewTarget = new ViewTarget<FutureStudioView, GlideDrawable>( customView ) {
        @Override
        public void onResourceReady(GlideDrawable resource, GlideAnimation<? super GlideDrawable> glideAnimation) {
            this.view.setImage( resource.getCurrent() );
        }
    };

    Glide
        .with( context.getApplicationContext() ) // safer!
        .load( eatFoodyImages[2] )
        .into( viewTarget );
}
```

### 转换transform
在图片显示之前，我们可以通过transform对图像做一些处理，达到我们想要的图片效果，例如我们改变图片的大小，范围，颜色等。Glide提供了两种基本的图片转换即：fitCenter 和 centerCrop。但有时我们想自定义转换效果，例如如果我们想展示一个圆形图片或者一个具有圆角的图片该如何处理？（圆形头像） 

为了自定义转换，我们需要创建一个新的类实现了 Transformation 接口。不过如果我们只是做图片的转换可以直接用Glide封装好的BitmapTransformation抽象类。图像转换操作只需要在transform里实现。getId() 方法描述了这个转换的唯一标识符。Glide 使用该键作为缓存系统的一部分，为了避免意外的问题，你要确保它是唯一的 
接下来先实现一个圆角的图片

```
/**
 * 将图像转换为四个角有弧度的图像
 */
public class GlideRoundTransform extends BitmapTransformation {
    private float radius = 0f;

    public GlideRoundTransform(Context context) {
        this(context, 100);
    }

    public GlideRoundTransform(Context context, int dp) {
        super(context);
        this.radius = Resources.getSystem().getDisplayMetrics().density * dp;
    }

    @Override
    protected Bitmap transform(BitmapPool pool, Bitmap toTransform, int outWidth, int outHeight) {
        return roundCrop(pool, toTransform);
    }

    private Bitmap roundCrop(BitmapPool pool, Bitmap source) {
        if (source == null) return null;

        Bitmap result = pool.get(source.getWidth(), source.getHeight(), Bitmap.Config.ARGB_8888);
        if (result == null) {
            result = Bitmap.createBitmap(source.getWidth(), source.getHeight(), Bitmap.Config.ARGB_8888);
        }
        Canvas canvas = new Canvas(result);
        Paint paint = new Paint();
        paint.setShader(new BitmapShader(source, BitmapShader.TileMode.CLAMP, BitmapShader.TileMode.CLAMP));
        paint.setAntiAlias(true);
        RectF rectF = new RectF(0f, 0f, source.getWidth(), source.getHeight());
        canvas.drawRoundRect(rectF, radius, radius, paint);
        Log.e("11aa", radius + "");
        return result;
    }

    @Override
    public String getId() {
        return getClass().getName() + Math.round(radius);
    }
}
Glide.with(context).load("http://img2.3lian.com/2014/f6/173/d/51.jpg").centerCrop().transform(new GlideRoundTransform(this,50)).animate(animator).into(imageView);
```

推荐一个开源的转换库[glide-transformations](https://github.com/wasabeef/glide-transformations)，它实现了很多转换，我们只要集成直接使用。

### Glide 的图片质量
在 Android 中有两个主要的方法对图片进行解码：ARGB8888（每像素4字节存储） 和 RGB565（每像素2字节存储）。当然ARGB8888有更高的图片质量，Glide默认使用RGB565进行解码，所以内存占用相对较小，如果我们想要更高的图片质量，可以设置，如下:  
`builder.setDecodeFormat(DecodeFormat.PREFER_ARGB_8888);`

### 内存缓存
Glide提供了一个类MemorySizeCalculator，用于决定内存缓存大小以及 bitmap 的缓存池。bitmap 池维护了你 App 的堆中的图像分配。正确的 bitmpa 池是非常必要的，因为它避免很多的图像重复回收，这样可以确保垃圾回收器的管理更加合理。它的默认计算实现:
```
//内存缓存
MemorySizeCalculator memorySizeCalculator = new MemorySizeCalculator(context);
int defaultMemoryCacheSize = memorySizeCalculator.getMemoryCacheSize();
int defalutBitmapPoolSize = memorySizeCalculator.getBitmapPoolSize();
```

也可以调整这个大小：
```
builder.setMemoryCache(new LruResourceCache((int) (defalutBitmapPoolSize * 1.2)));//内部
        builder.setBitmapPool(new LruBitmapPool((int) (defalutBitmapPoolSize * 1.2)));
```

也可以设置不使用内存缓存，调用方法skipMemoryCache(true)
```
Glide.with(this)
    .load("http://ww4.sinaimg.cn/large/610dc034gw1f96kp6faayj20u00jywg9.jpg")
    .skipMemoryCache(true)
    .into(image);
```

### 磁盘缓存
Glide图片缓存有两种情况，一种是内部磁盘缓存另一种是外部磁盘缓存。我们可以通过 builder.setDiskCache（）设置，并且Glide已经封装好了两个类实现外部和内部磁盘缓存，分别是InternalCacheDiskCacheFactory和ExternalCacheDiskCacheFactory。
```
//磁盘缓存100M
builder.setDiskCache(new InternalCacheDiskCacheFactory(context, 1024 * 1024 * 100));//内部磁盘缓存
builder.setDiskCache(new ExternalCacheDiskCacheFactory(context, 100 * 1024 * 1024));//磁盘缓存到外部存储
```

磁盘缓存策略
```
DiskCacheStrategy.NONE: 不使用硬盘缓存
DiskCacheStrategy.SOURCE: 将原始图像缓存在硬盘中
DiskCacheStrategy.RESULT: 将显示出来大小的图像缓存在硬盘(默认缓存策略)
DiskCacheStrategy.ALL: 显示的图像和原始图像都会缓存
```
缓存策略通过方法diskCacheStrategy()来设定
```
Glide.with(this)
    .load("http://ww4.sinaimg.cn/large/610dc034gw1f96kp6faayj20u00jywg9.jpg")
    .diskCacheStrategy(DiskCacheStrategy.NONE) //不使用硬盘缓存
    .into(image);
```