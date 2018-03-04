---
layout:     post
title:      "EffectiveJava读书笔记-第二章创建和销毁对象（1-7）"
date:       2016-09-12 21:48:06
author:     "afayp"
catalog:    true
tags:
    - Java
    - 读书笔记
---



# 第一条 考虑用静态工厂方法代替构造器
静态工厂方法是一个返回类的实例的静态方法,对它其实只是一个普通的静态方法而已,需要注意的是它与设计模式中的工厂方法不同,不要弄混淆了。

<!--more-->


Java源码实例：
```java
public static Boolean valueOf(boolean b){
	return b?Boolean.TRUE:Boolean.FALSE;
}
```

静态工厂方法与构造器相比,有哪些优势？

1. 有名称
	构造方法没有名称,给静态工厂取一个适当的方法名称会更易于阅读,也更易于客户端使用，特别是在面对有多个参数或者多个构造方法的类的时候,会让客户端不知道如何选择构造方法。当一个类需要多个带有相同签名的构造器时，就用静态工厂方法代替构造器，并选择好名称突出他们之间的区别。
2. 不必每次调用它们的时候都创建一个新的对象
	我们知道构造方法每次都会创建一个新的实例,而当我们需要避免重复创建不必要的对象(或者不可变的类)时通过构造方法去做是做不到的,而静态方法可以为重复的调用返回相同的对象(比如返回的对象需要是一个单例)。另外如果创建对象的成本很高，则静态工厂方法会很有优势。
3. 可以返回原类型的任何子类对象
	构造方法只能返回类本身,而静态方法可以返回它的子类,利用多态的特性使得静态方法更加灵活
	比如Executors类的各种newxxx方法返回ExecutorService的子类
4. 在参数化类型实例时，使得代码更加简洁
	Jdk7已经实现了类型推到，所以这点没什么意义

劣势？

1. 类如果不含公有的或者受保护的构造器，就不能被子类化。
	这条不懂？
2. 它与其他的静态方法没有任何区别
	构造器会在API文档中明确标出，但静态方法真的只是一个静态的方法而已,如果文档上不提到创建该类的实例可以使用静态方法的时候,很容易被忽略
	静态方法推荐的标准命名：

> - valueOf—返回的实例和它的参数具有相同的值，一般用于类型转换，如String，Boolean，Double等
- of—valueOf的简洁写法。
- getInsatance—返回的实例通过方法的参数决定，对于单例一般没有参数，并且每次返回唯一实例
- newInstance—每次返回不同的实例
- getType—和getInstance类似，在工厂方法处于不同的类中的时候使用，Type表示工厂方法所返回的对象类型。
- newType—和getType类似，在工厂方法处于不同的类的时候使用，Type表示工厂方法所返回的对象类型

总结
静态工厂方法通常更加合适，优先考虑静态工厂方法。比如Fragment中。


# 第二条 遇到多个构造器参数时要考虑用构建器
当实例化一个类时有很多可选参数的时候,我们或许会写很多不同参数的构造方法,可读性会非常差,并且客户端也不知道在什么时候用什么构造器,这是非常非常糟糕的,同样静态方工厂也不能避免,这个时候我们需要用到构建器,也即设计模式里的Builder模式。不直接生成对象，而是让客户端利用必要参数调用构造器得到一个builder对象，然后在builder对象上调用类似于setter的方法来设置每个相关的可选参数，最后客户端调用无参的build方法来生成对象。其中这个builder是它构建的类的静态成员类。
书里面的例子：
```java
	public class NutritionFacts {
	    private final int servingSize;
	    private final int servings;
	    private final int calories;
	    private final int fat;
	 
	    
	    public static class Builder{
	        //必要参数
	        private final int servingSizes;
	        private final  int servings;
	        //可选参数,有默认值
	        private int calories = 0;
	        private int fat = 0;
	        
	        //构造中传入必要参数
	        public Builder(int servingSizes,int servings){
	            this.servingSizes = servingSizes;
	            this.servings = servings;
	        }
	        
	        //设置可选参数的方法，return自身，以便可以把调用链接起来。
	        public Builder calories(int val){
	            calories = val;
	            return this;
	        }
	        public Builder fat(int val){
	            fat = val;
	            return this;
	        }
	        //最后调用build方法来生成对象
	        public NutritionFacts build(){
	            return new NutritionFacts(this);
	        }
	    }
	    //构造方法私有化，只能通过build来生成需要的对象
	    private NutritionFacts(Builder builder) {
	        servingSize = builder.servingSizes;
	        servings = builder.servings;
	        calories = builder.calories;
	        fat = builder.fat;
	    }
	}
	
客户端代码：
	NutritionFacts nutritionFacts = new NutritionFacts.Builder(240,8)
	        .calories(100)
	        .fat(35)
	        .build();
```
build模式模拟了可选参数，就像Python中的一样。客户端的代码更易编写和阅读。
不足：为了创建一个对象，必须先创建Builder，虽然开销并不明显，但在某些十分注重性能的情况下可能就会有问题。
总结：当类的构造器或者静态工厂中具有多个参数时（4个或以上），可以选择`Builder`模式，特别是当大多数参数为可选时。

# 第三条 用私有的构造器或者枚举类型强化Singleton属性
·创建单例的几种方法就不提了。还有如何防反射（修改构造器，在第二次创建对象时抛出异常），防反序列化，看不懂~
用枚举实现单例（表示并不常见。。）

# 第四条 通过私有构造器强化不可实例化的能力
有些工具类只包含静态方法和静态域（static-field），不希望被实例化，实例对它来说没有任何意义，然而在缺少显示构造器的情况下，编译器会自动提供一个共有的，无参的缺省构造器。如果不希望该类被实例化，该怎么办？提供私有构造器，并在里面抛出异常：
```java
public class Util{
  private Util{
   // Suppress default constructor for noninstantiability
   throw new AssertionError();
  }
}
```
可能会觉得有点变扭，但实际上很多优秀的库都是这么写的，比如`RxJava`中的`Subscriptions`类:
```java
	public final class Subscriptions {
	    private Subscriptions() {
	        throw new IllegalStateException("No instances!");
	    }
	}
```

# 第五条 避免创建不必要的对象
最好能重用对象而不是在每次需要的时候创建一个相同功能的新对象.如果对象是不可变的(immutable),它就始终可以被重用。

- 比如 `String s = new String("stringette")` 这样的写法非常糟糕，因为参数`“stringtte”`本身就是一个`String`实例，功能方面等同于创建出来的对象。
改进：`String s = "stringette"` 只用了一个String实例，并且利用常量池该对象可以被重用。
- 对于同时提供了静态工厂方法和构造器的不可变类，通常使用静态工厂方法而不是构造器，以免重复创建不必要的对象。
例如`Boolea`n类,提供了`TRUE`和`FALS`E两个`final`对象
`public static final Boolean TRUE = new Boolean(true);`
`public static final Boolean FALSE = new Boolean(false);`
我们总是用`Boolean.valueOf(true)`这个静态工厂方法来创建`Boolean`对象，而不是new出来，因为前者重用了`FALS`E和`TRUE`来避免重复创建相同功能的对象。
- 如果一个对象需要重复多次使用，并且这个对象是不可变的或者说是可变的但我们不会去改变它，这种情况下可以把该对象设置为`static final`，避免重复创建（可以一开始就new出来，也可以放在static块中）
- 优先使用基本类型，而不是装箱基本类型，并且要当心无意识的装箱。
- 对象池：维护对象池来避免创建对象只针对非常重量级的对象,如数据库连接池，`Android`中对象池有很多,如`Message`类,又如`Glide`中的`Bitmap`池
对象池的缺点： 代码更乱 涉及到回收、重用必然会增加许多代码；增加内存占用,损害性能
- 本条并不是说创建对象代价是昂贵的，要尽可能的避免创建对象，相反小对象的创建和回收是非常廉价的特别是现在JVM性能这么高。通过创建对象使程序更加清晰简洁通常是件好事。但是如果创建对象代价非常昂贵，比如数据库连接，那么重用这些对象非常有意义。
	
# 第六条 消除过期的对象引用
虽然Java有自动GC，但我们还是要考虑内存管理。
书里面介绍的三种内存泄露的情况：

1. 当一个数组扩容后又缩减,比如size从0->200->100(一个栈先增长,后收缩),那么从栈中弹出的对象（index>=100的对象）不会被当做垃圾回收，即使程序不再引用这些对象。因为栈内部维护着这些对象的过期引用（obsolete reference）。所谓的过期引用是指永远不会被解除的引用。
	解决方法：一旦对象引用已经过期，只需清空这些引用即可（置为null）
	不是说对于每个对象引用我们都需要手动清空，这没有必要。只有对于那些自己管理内存的类，我们才需要这么做（？）
2.  内存泄露另一个常见来源是缓存。
	解决方法：看不懂。。。
3.  监听器和其他回调
	如果你实现了一个API，客户端在这个API中注册回调，却没有显示的取消注册，那么它们就会积聚。
	解决方法：看不懂。。。
	
# 第七条 避免使用终结方法finalizer
本来就不会用啊( ⊙ o ⊙ )！

	


	 
	




