---
layout:     post
title:      "Android Progressbar知识点"
date:       2016-04-01 17:12:02
author:     "afayp"
catalog:    true
tags:
    - Android
---


  
# 分类
分三种形式：

1. 在xml布局文件中定义
2. 直接request(标题进度条)
3. 对话框形式（不用在布局文件中定义）


<!--more-->

# 显示风格

大的环形progressBar: style="?android:attr/progressBarStyleLarge"  
中的环形progressBar: style不设置  
小的环形progressBar： style="?android:attr/progressBarStyleSmall"  
水平progressBar： style="?android:attr/progressBarStyleHorizontal"


# 标题进度条
```java
 	// 启用窗口特征，启用带进度和不带进度的进度条（不用在布局文件中写，直接用！）
    requestWindowFeature(Window.FEATURE___PROGRESS);
    requestWindowFeature(Window.FEATURE_INDETERMINATE_PROGRESS);
    setContentView(R.layout.main);//要在之后

    // 显示两种进度条,true显示；false则不显示
    setProgressBarVisibility(true);
    setProgressBarIndeterminateVisibility(false);
    // 最大值Max=10000，一般设置9999
    setProgress(9999);

```
 
# 布局中定义progressbar

## 常用属性

```java
	<ProgressBar
        android:id="@+id/horiz"
        style="@android:style/Widget.ProgressBar.Horizontal"
        android:progressDrawable="@drawable/progress_bar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:max="100"//最大显示进度
        android:progress="50"//第一显示进度
        android:secondaryProgress="80" //第二显示进度
        android:indeterminate = “true” //设置是否精确显示（★注：true表示不精确显示，false表示精确显示进度）
        />

```

## 常用方法
1. setProgress(int) 设置第一进度

2. setSecondaryProgress(int) 设置第二进度

3. getProgress( ) 获取第一进度

4. getSecondaryProgress( ) 获取第二进度

5. incrementProgressBy(int) 增加或减少第一进度

6. incrementSecondaryProgressBy(int) 增加或减少第二进度

7. getMax( )获取最大进度

8. setVisibility(int v)：设置该进度条是否可视 

9. isIndeterminate()：指示进度条是否在不确定模式下

10. setIndeterminate(boolean indeterminate)：设置不确定模式下


# progressDialog 
```java
			//新建ProgressDialog对象
			prodialog=new ProgressDialog(MainActivity.this);
            //设置显示风格
            prodialog.setProgressStyle(ProgressDialog.STYLE_HORIZONTAL);
            //设置标题
            prodialog.setTitle("幕课网");
            //设置对话框里的文字信息
            prodialog.setMessage("欢迎大家支持幕课网");
            //设置图标
            prodialog.setIcon(R.drawable.ic_launcher);
            
            /**
             * 设定关于ProgressBar的一些属性
             */
            //设定最大进度
            prodialog.setMax(100);
            //设定初始化已经增长到的进度
            prodialog.incrementProgressBy(50);
            //进度条是否明确显示进度（false为明确）
            prodialog.setIndeterminate(false);
            //是否可以通过返回按钮退出对话框
            prodialog.setCancelable(true);
            
            /**
             * 设定一个确定按钮
             */
            //参数：按钮类型；文字；监听器
            prodialog.setButton(DialogInterface.BUTTON_POSITIVE, "确定", new DialogInterface.OnClickListener() {
                
                @Override
                public void onClick(DialogInterface dialog, int which) {
                }
            });
            //显示ProgressDialog
            prodialog.show();//prodialog.dismiss();
           
```


# 自定义progressbar样式
见参考链接

# 参考链接

<http://blog.csdn.net/mad1989/article/details/38042875>