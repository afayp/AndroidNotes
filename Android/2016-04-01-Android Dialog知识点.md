---
layout:     post
title:      "Android Dialog"
date:       2016-04-01 13:14:35
author:     "afayp"
catalog:    true
tags:
    - Android
---


#分类
对话框一般是一个出现在当前Activity之上的一个小窗口. 处于下面的Activity失去焦点, 对话框接受所有的用户交互。
<!--more-->
分类：  

- 警告对话框 AlertDialog:  一个可以有0到3个按钮, 一个单选框或复选框的列表的对话框. 警告对话框可以创建大多数的交互界面, 是推荐的类型.
- 进度对话框 ProgressDialog:  显示一个进度环或者一个进度条. 由于它是AlertDialog的扩展, 所以它也支持按钮.
- 日期选择对话框 DatePickerDialog:  让用户选择一个日期.
- 时间选择对话框 TimePickerDialog:  让用户选择一个时间.
- 自定义的对话框



# AlertDialog
AlertDialog的构造方法全部是Protected的，所以不能直接通过new一个AlertDialog来创建出一个AlertDialog。
要创建一个AlertDialog，就要用到AlertDialog.Builder中的create()方法。

使用AlertDialog.Builder创建对话框需要了解以下几个方法：

- setTitle ：为对话框设置标题
- setIcon ：为对话框设置图标
- setMessage：为对话框设置内容
- setView ： 给对话框设置自定义样式
- setItems ：设置对话框要显示的一个list，一般用于显示几个命令时
- setMultiChoiceItems ：用来设置对话框显示一系列的复选框
- setNeutralButton    ：普通按钮
- setPositiveButton   ：给对话框添加"Yes"按钮
- setNegativeButton ：对话框添加"No"按钮
- create ： 创建对话框
- show ：显示对话框
- dismiss：关闭对话框
- setCancelable(boolean)将对话框设为不可取消（不能使用back键来取消）

虽然实际开发中很少使用系统的dialog，不过下面还是介绍一下。
## 例子

### 简单的AlertDialog
```java
Dialog alertDialog = new AlertDialog.Builder(this).   
setTitle("对话框的标题").   
setMessage("对话框的内容").   
setIcon(R.drawable.ic_launcher)
.create();   
alertDialog.show();  
```

### 带按钮的AlertDialog
```java
Dialog alertDialog = new AlertDialog.Builder(this)
				.setTitle("确定删除？").   
                .setMessage("您确定删除该条信息吗？").   
                .setIcon(R.drawable.ic_launcher).   
                .setPositiveButton("确定", new DialogInterface.OnClickListener() {   
                        
                     @Override   
                    public void onClick(DialogInterface dialog, int which) {   
                        // dialog.dismiss();  
                     }   
                 }).   
                setNegativeButton("取消", new DialogInterface.OnClickListener() {   
                        
                    @Override   
                     public void onClick(DialogInterface dialog, int which) {   
                         // dialog.dismiss();  
                     }   
                 }).   
                 setNeutralButton("查看详情", new DialogInterface.OnClickListener() {   
                        
                    @Override   
                    public void onClick(DialogInterface dialog, int which) {   
                         // TODO Auto-generated method stub    
                     }   
                 }).   
                 create();   
        alertDialog.show();   
```
如果嫌写三个监听器麻烦，也可以这么写：
```java
 //先new出一个监听器，设置好监听  
       DialogInterface.OnClickListener dialogOnclicListener=new DialogInterface.OnClickListener(){  
  
           @Override  
           public void onClick(DialogInterface dialog, int which) {  
               switch(which){  
                   case Dialog.BUTTON_POSITIVE:  
                       Toast.makeText(MainActivity.this, "确认" + which, Toast.LENGTH_SHORT).show();  
                       break;  
                   case Dialog.BUTTON_NEGATIVE:  
                       Toast.makeText(MainActivity.this, "取消" + which, Toast.LENGTH_SHORT).show();  
                       break;  
                   case Dialog.BUTTON_NEUTRAL:  
                       Toast.makeText(MainActivity.this, "忽略" + which, Toast.LENGTH_SHORT).show();  
                       break;  
               }  
           }  
       };  
       //dialog参数设置  
       AlertDialog.Builder builder=new AlertDialog.Builder(this);  //先得到构造器  
       builder.setTitle("提示"); //设置标题  
       builder.setMessage("是否确认退出?"); //设置内容  
       builder.setIcon(R.mipmap.ic_launcher);//设置图标，图片id即可  
       builder.setPositiveButton("确认",dialogOnclicListener);  
       builder.setNegativeButton("取消", dialogOnclicListener);  
       builder.setNeutralButton("忽略", dialogOnclicListener);  
       builder.create().show();  
```



### 带列表的AlertDialog
```java
//用到这个API：setItems(CharSequence[] items, final OnClickListener listener)
String[] arrayFruit = new String[] { "苹果", "橘子", "草莓", "香蕉" };   
Dialog alertDialog = new AlertDialog.Builder(this) 
                .setTitle("你喜欢吃哪种水果？")   
                .setIcon(R.drawable.ic_launcher)   
                .setItems(arrayFruit, new DialogInterface.OnClickListener() {   
    
                    @Override   
                    public void onClick(DialogInterface dialog, int which) {   
                        Toast.makeText(Dialog_AlertDialogDemoActivity.this, arrayFruit[which], Toast.LENGTH_SHORT).show();   
                    }   
                }) 
                .setNegativeButton("取消", new DialogInterface.OnClickListener() {   
   
                    @Override   
                    public void onClick(DialogInterface dialog, int which) {   
                        // TODO Auto-generated method stub    
                    }   
                })
                .create();   
alertDialog.show();   

```
### 带RadioButton的AlertDialog（单选）

用到：setSingleChoiceItems(CharSequence[] items, int checkedItem, final OnClickListener listener)
第一个参数是要显示的数据的数组，第二个参数是初始值（初始被选中的item，使用“-1”来表示默认情况下不选中任何选项。），第三个参数是点击某个item的触发事件
```java
String[] arrayFruit = new String[] { "苹果", "橘子", "草莓", "香蕉" };   
Dialog alertDialog = new AlertDialog.Builder(this)  
                .setTitle("你喜欢吃哪种水果？")
                .setIcon(R.drawable.ic_launcher)   
                .setSingleChoiceItems(arrayFruit, 0, new DialogInterface.OnClickListener() {   
    
                    @Override   
                    public void onClick(DialogInterface dialog, int which) {   
                        selectedFruitIndex = which;   
                    }   
                })
                .setPositiveButton("确认", new DialogInterface.OnClickListener() {   
   
                    @Override   
                    public void onClick(DialogInterface dialog, int which) {   
                        Toast.makeText(Dialog_AlertDialogDemoActivity.this, arrayFruit[selectedFruitIndex], Toast.LENGTH_SHORT).show();   
                    }   
                })
                .setNegativeButton("取消", new DialogInterface.OnClickListener() {   
   
                    @Override   
                    public void onClick(DialogInterface dialog, int which) {   
                        // TODO Auto-generated method stub    
                    }   
                })  
                .create();   
alertDialog.show(); 

```

### 带checkbox的AlertDialog（多选）
用到：setMultiChoiceItems(CharSequence[] items, boolean[] checkedItems, final OnMultiChoiceClickListener listener)  
第一个参数是要显示的数据的数组，第二个参数是选中状态的数组，第三个参数是点击某个item的触发事件
```java
String[] arrayFruit = new String[] { "苹果", "橘子", "草莓", "香蕉" };   
boolean[] arrayFruitSelected = new boolean[] {true, true, false, false};   
Dialog alertDialog = new AlertDialog.Builder(this)  
                .setTitle("你喜欢吃哪种水果？") 
                .setIcon(R.drawable.ic_launcher)   
                .setMultiChoiceItems(arrayFruit, arrayFruitSelected, new DialogInterface.OnMultiChoiceClickListener() {   
                       
                    @Override   
                    public void onClick(DialogInterface dialog, int which, boolean isChecked) {   
                        arrayFruitSelected[which] = isChecked;   
                    }   
                }) 
                .setPositiveButton("确认", new DialogInterface.OnClickListener() {   
   
                    @Override   
                    public void onClick(DialogInterface dialog, int which) {   
                        StringBuilder stringBuilder = new StringBuilder();   
                        for (int i = 0; i < arrayFruitSelected.length; i++) {   
                            if (arrayFruitSelected[i] == true)   
                            {   
                                stringBuilder.append(arrayFruit[i] + "、");   
                            }   
                        }   
                        Toast.makeText(Dialog_AlertDialogDemoActivity.this, stringBuilder.toString(), Toast.LENGTH_SHORT).show();   
                    }   
                })
                .setNegativeButton("取消", new DialogInterface.OnClickListener() {   
   
                    @Override   
                    public void onClick(DialogInterface dialog, int which) {   
                        // TODO Auto-generated method stub    
                    }   
                })  
                .create();   
alertDialog.show();   

```

### 自定义view的AlertDialog

#### setContentView(int)
标题用自带的。

```java
Dialog dialog = new Dialog(mContext);   
dialog.setContentView(R.layout.custom_dialog);  
dialog.setTitle("Custom Dialog");  
TextView text = (TextView) dialog.findViewById(R.id.text);  
text.setText("Hello, this is a custom dialog!");  
ImageView image = (ImageView) dialog.findViewById(R.id.image);  
image.setImageResource(R.drawable.android);  
```

#### setView(View)
全部用自定义view显示

```java
// 填充自定义View    
LayoutInflater layoutInflater = LayoutInflater.from(this);   
View myLoginView = layoutInflater.inflate(R.layout.login, null);   
Dialog alertDialog = new AlertDialog.Builder(this) 
                .setTitle("用户登录")
                .setIcon(R.drawable.ic_launcher)  
                .setView(myLoginView).   
                .setPositiveButton("登录", new DialogInterface.OnClickListener() {   
   
                    @Override   
                    public void onClick(DialogInterface dialog, int which) {   
                        // TODO Auto-generated method stub    
                    }   
                }) 
                .setNegativeButton("取消", new DialogInterface.OnClickListener() {   
   
                    @Override   
                    public void onClick(DialogInterface dialog, int which) {   
                        // TODO Auto-generated method stub    
                    }   
                })
                .create();   
alertDialog.show();   

```

## 改变AlertDialog大小，位置
```java
AlertDialog dialog = new AlertDialog.Builder(this).create();
dialog.show();
WindowManager.LayoutParams params = dialog.getWindow().getAttributes();
params.width = 200;//宽
params.height = 200 ;//高
dialog.getWindow().setAttributes(params);
//改变位置 
dialog.getWindow().setLayout(300, 200);  
//dialog.show();一定要放在dialog.getWindow().setLayout(300, 200);的前面，否则不起作用
//去除边框
AlertDialog.setView(view,0,0,0,0);

```

# ProgressDialog

# 参考链接
<http://blog.csdn.net/hgl868/article/details/6738512>

