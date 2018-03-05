---
layout:     post
title:      "ContentProvider知识点"
date:       2016-06-07 14:03:39
author:     "afayp"
catalog:    true
tags:
    - Android
---



# 定义
ContentProvider：为存储和获取数据提供统一的接口。可以在不同的应用程序之间共享数据。其实它也只是一个中间人，真正的数据源是文件或者SQLite等。

<!--more-->

ContentProvider有两种用法：

- 一种是使用现有的ContentProvider来读取需要的数据
- 另一种是创建自己的ContentProvider给我们的程序提供外部访问的接口


# ContentResolver

ContentResolver跟ContentProvider是对应的关系，我们通过它来与ContentProvider进行数据交换的。android.content.Context类为我们定义了getContentResolver()方法，用于获取一个ContentResolver对象，如果我们在运行期可以通过getContext()获取当前Context实例对象，就可以通过这个实例对象所提供的getContentResolver()方法获取到ContentResolver类型的实例对象，进而可以操作对应的数据。


ContentResolver通过内容URI来进行CRUD操作。内容URI为ContentProvider里面的数据建立了唯一的标识符。

# URI


Uri主要包含了两部分信息：1.需要操作的ContentProvider ，2.对ContentProvider中的什么数据进行操作.

![](http://img.my.csdn.net/uploads/201103/1/0_1298993159Mwmo.gif)

一个Uri由以下几部分组成：

1. scheme：ContentProvider（内容提供者）的scheme已经由Android所规定为：content://。

2. 主机名（或Authority）：用于唯一标识这个ContentProvider，外部调用者可以根据这个标识来找到它。一般都是该ContentProvider的完整包名。

3. 路径（path）：可以用来表示我们要操作的数据，路径的构建应根据业务而定（一般都是：表名/特定的记录），如下：
	- 要操作contact表中id为10的记录，可以构建这样的路径:/contact/10
	- 要操作contact表中id为10的记录的name字段， contact/10/name
	- 要操作contact表中的所有记录，可以构建这样的路径:/contact（后面没有id就表示返回整张表）
	- "content://com.android.calendar/calendars/#" #表示数据id（#代表任意数字）
	- "content://com.android.calendar/calendars/*" *来匹配任意文本
	- 要操作的数据不一定来自数据库，也可以是文件等他存储方式，如下:
	- 要操作xml文件中contact节点下的name节点，可以构建这样的路径：/contact/name
	
如果要把一个字符串转换成Uri，可以使用Uri类中的parse()方法，如下：
```java
Uri uri = Uri.parse("content://com.changcheng.provider.contactprovider/contact")
```
## URI,URL,URN区别联系
![](http://images2015.cnblogs.com/blog/591228/201601/591228-20160116223301225-1866838315.png)

“URI可以分为URL,URN或同时具备locators 和names特性的一个东西。URN作用就好像一个人的名字，URL就像一个人的地址。换句话说：URN确定了东西的身份，URL提供了找到它的方式。”

URL是一种具体的URI，它不仅唯一标识资源，而且还提供了定位该资源的信息。URI是一种语义上的抽象概念，可以是绝对的，也可以是相对的，而URL则必须提供足够的信息来定位，所以，是绝对的，而通常说的relative URL，则是针对另一个absolute URL，本质上还是绝对的。  
注：这里的绝对(absolute)是指包含scheme，而相对(relative)则不包含scheme。

URL 就是 URI 的 定位。
但URI 不一定是 URL。


#CRUD
有了URI之后就可以通过ContentResolver来进行CRUD操作了

## 查询



# 创建ContentProvider
