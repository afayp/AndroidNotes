---
title: Java面试题
date: 2017-01-18 23:49:02
categories: Java
tags: Java
---
一些常见的基础Java面试题
<!--more>

- 什么是B/S架构？什么是C/S架构？ 
>   
B/S(Browser/Server)，浏览器/服务器程序  
C/S(Client/Server)，客户端/服务端，桌面应用程序

- Java的数据结构有那些？
> 
线性表（ArrayList）  
链表（LinkedList）  
栈（Stack）  
队列（Queue）  
图（Map）  
树（Tree）  

- 类与对象的关系?
> 类是对象的抽象，对象是类的具体，类是对象的模板，对象是类的实例

- Java中有几种基本数据类型
> 
整形：byte,short,int,long  
浮点型：float,double  
字符型：char  
布尔型：boolean  

- Java中的包装类都是那些？
> 
byte：Byte
short：Short
int：Integer
long：Long
float：Float
double：Double
char：Character
boolean：Boolean

- 什么是拆装箱？
> 
拆箱：把包装类型转成基本数据类型
装箱：把基本数据类型转成包装类型

- Java常用包有那些？
> 
Java.lang  
Java.io  
Java.sql  
Java.util  
Java.awt  
Java.net  
Java.math  

- Static关键字有什么作用？
> 
Static可以修饰内部类、方法、变量、代码块  
Static修饰的类是静态内部类  
Static修饰的方法是静态方法，表示该方法属于当前类的，而不属于某个对象的，静态方法也不能被重写，可以直接使用类名来调用。在static方法中不能使用this或者super关键字。  
Static修饰变量是静态变量或者叫类变量，静态变量被所有实例所共享，不会依赖于对象。静态变量在内存中只有一份拷贝，在JVM加载类的时候，只为静态分配一次内存。   
Static修饰的代码块叫静态代码块，通常用来做程序优化的。静态代码块中的代码在整个类加载的时候只会执行一次。静态代码块可以有多个，如果有多个，按照先后顺序依次执行。

- Final在java中的作用
> 
1.被final修饰的类不可以被继承   
2.被final修饰的方法不可以被重写   
3.被final修饰的变量不可以被改变.如果修饰引用,那么表示引用不可变,引用指向的内容可变.  
4.被final修饰的方法,JVM会尝试将其内联,以提高运行效率   
5.被final修饰的常量,在编译阶段会存入常量池中. 

- String,StringBuffer和StringBuilder区别
> 
String是字符串常量,final修饰;StringBuffer字符串变量(线程安全); StringBuilder 字符串变量(线程不安全).
>
- String和StringBuffer
>
String和StringBuffer主要区别是性能:String是不可变对象,每次对String类型进行操作都等同于产生了一个新的String对象,然后指向新的String对象.所以尽量不在对String进行大量的拼接操作,否则会产生很多临时对象,导致GC开始工作,影响系统性能.
> 
StringBuffer是对对象本身操作,而不是产生新的对象,因此在有大量拼接的情况下,我们建议使用StringBuffer.
> 
但是需要注意现在JVM会对String拼接做一定的优化: String s=“This is only ”+”simple”+”test”会被虚拟机直接优化成String s=“This is only simple test”,此时就不存在拼接过程.

- StringBuffer，Stringbuilder有什么区别？
> 
StringBuffer与StringBuilder都继承了AbstractStringBulder类，而AbtractStringBuilder又实现了CharSequence接口，两个类都是用来进行字符串操作的。
在做字符串拼接修改删除替换时，效率比string更高。  
StringBuffer是线程安全的，Stringbuilder是非线程安全的。所以Stringbuilder比stringbuffer效率更高，StringBuffer的方法大多都加了synchronized关键字

- String str=”aaa”,与String str=new String(“aaa”)一样吗？
> 
不一样的。因为内存分配的方式不一样。  
第一种，创建的”aaa”是常量，jvm都将其分配在常量池中。  
第二种创建的是一个对象，jvm将其值分配在堆内存中。

- String类的常用方法有那些？
>  
charAt：返回指定索引处的字符  
indexOf()：返回指定字符的索引  
replace()：字符串替换  
trim()：去除字符串两端空白  
split()：分割字符串，返回一个分割后的字符串数组  
getBytes()：返回字符串的byte类型数组  
length()：返回字符串长度  
toLowerCase()：将字符串转成小写字母  
toUpperCase()：将字符串转成大写字符  
substring()：截取字符串  
format()：格式化字符串  
equals()：字符串比较  

- 面向对象的语言有那些特征？
> 
封装、继承、多态

- 什么是重写？什么是重载？
> 
重载和重写都是java多态的表现。  
重载叫override，在同一个类中多态的表现。当一个类中出现了多个相同名称的方法，但参数个数和参数类型不同，方法重载与返回值无关  
重写叫overwrite，是字符类中多态的表现。当子类出现与父类相同的方法，那么这就是方法重写。方法重写时，子类的返回值必须与父类的一致。如果父类方法抛出一个异常，子类重写的方法抛出的异常类型不能小于父类抛出的异常类型。

- 抽象类必须要有抽象方法吗
> 
不是必须。抽象类可以没有抽象方法。但包含抽象方法的类一定是抽象类

- 抽象类可以使用final修饰吗？
> 
不可以。定义抽象类就是让其他继承的，而final修饰类表示该类不能被继承，与抽象类的理念违背了

- 接口有什么特点？
> 
接口中声明全是public static final修饰的常量  
接口中所有方法都是抽象方法  
接口是没有构造方法的  
接口也不能直接实例化  
接口可以多继承  

- Java中异常分为哪两种？
> 
编译时异常  
运行时异常

- 常见异常
> NullPointerException：空指针异常  
ArrayIndexOutOfBoundsException：数组下标越界  
NumberFormatException：数字转换异常  
IllegalArgumentException：参数不匹配异常  
InstantiationException：对象初始化异常  
ArithmeticException：算术异常   

- java 创建对象的几种方式
> 
采用new  
通过反射  
采用clone  
通过序列化机制  

- Java的io流分为哪两种？
> 
按功能来分:
输入流(input)，输出流(output)  
按类型来分:
字节流，字符流

- 常用io类有那些？
> 
File  
FileInputSteam，FileOutputStream  
BufferInputStream，BufferedOutputSream  
PrintWrite  
FileReader，FileWriter  
BufferReader，BufferedWriter  
ObjectInputStream，ObjectOutputSream  

- 字节流与字符流的区别
> 
以字节为单位输入输出数据，字节流按照8位传输
以字符为单位输入输出数据，字符流按照16位传输

- final、finalize()、finally
> 
性质不同  
final为关键字；
finalize()为方法；
finally为区块标志，用于try语句中；
>
作用  
final为用于标识常量的关键字，final标识的关键字存储在常量池中（在这里final常量的具体用法将在下面进行介绍）；  
finalize()方法在Object中进行了定义，用于在对象“消失”时，由JVM进行调用用于对对象进行垃圾回收，类似于C++中的析构函数；用户自定义时，用于释放对象占用的资源（比如进行I/0操作）；  
finally{}用于标识代码块，与try{}进行配合，不论try中的代码执行完或没有执行完（这里指有异常），该代码块之中的程序必定会进行；  

- 什么是不可变对象
> 
不可变对象指对象一旦被创建，状态就不能再改变。任何修改都会创建一个新的对象，如 String、Integer及其它包装类。

- 静态变量和实例变量的区别?
> 
静态变量存储在方法区,属于类所有.实例变量存储在堆当中,其引用存在当前线程栈.

- String s1="ab",String s2="a"+"b",String s3="a",String s4="b",s5=s3+s4请问s5==s2返回什么?
> 
返回false.在编译过程中,编译器会将s2直接优化为"ab",会将其放置在常量池当中,s5则是被创建在堆区,相当于s5=new String("ab");

- Object中有哪些公共方法?
> 
equals()  
clone()  
getClass()  
notify(),notifyAll(),wait()  
toString  

- 为什么要有不同的引用类型
> 
不像C语言,我们可以控制内存的申请和释放,在Java中有时候我们需要适当的控制对象被回收的时机,因此就诞生了不同的引用类型,可以说不同的引用类型实则是对GC回收时机不可控的妥协.有以下几个使用场景可以充分的说明:
>
利用软引用和弱引用解决OOM问题：用一个HashMap来保存图片的路径和相应图片对象关联的软引用之间的映射关系，在内存不足时，JVM会自动回收这些缓存图片对象所占用的空间，从而有效地避免了OOM的问题.
通过软引用实现Java对象的高速缓存:比如我们创建了一Person的类，如果每次需要查询一个人的信息,哪怕是几秒中之前刚刚查询过的，都要重新构建一个实例，这将引起大量Person对象的消耗,并且由于这些对象的生命周期相对较短,会引起多次GC影响性能。此时,通过软引用和 HashMap 的结合可以构建高速缓存,提供性能.

- short s1= 1; s1 = s1 + 1; 该段代码是否有错,有的话怎么改？
>
有错误,short类型在进行运算时会自动提升为int类型,也就是说s1+1的运算结果是int类型.

- short s1= 1; s1 += 1; 该段代码是否有错,有的话怎么改？
>
+=操作符会自动对右边的表达式结果强转匹配左边的数据类型,所以没错.

- 一个.java文件内部可以有多个类?(非内部类)
> 
只能有一个public公共类,但是可以有多个default修饰的类.

- java中int char,long各占多少字节?

|类型|位数|字节数| 
|-|-|-| 
|short|2|16| 
|int|4|32| 
|long|8|64| 
|float|4|32 
|double|8|64| 
|char|2|16|

- 64位的JVM当中,int的长度是多少?
> 
Java 中，int 类型变量的长度是一个固定值，与平台无关，都是 32 位。意思就是说，在 32 位 和 64 位 的Java 虚拟机中，int 类型的长度是相同的。

- 如何将byte转为String
>
可以使用 String 接收 byte[] 参数的构造器来进行转换，需要注意的点是要使用的正确的编码，否则会使用平台默认编码，这个编码可能跟原来的编码相同，也可能不同。

- 你知道哪些垃圾回收算法?
> 
标记-清除  
标记-复制  
标记-整理  
分代回收  

- 如何判断一个对象是否应该被回收
> 
这就是所谓的对象存活性判断,常用的方法有两种:1.引用计数法;2:对象可达性分析.由于引用计数法存在互相引用导致无法进行GC的问题,所以目前JVM虚拟机多使用对象可达性分析算法.

- 简单的解释一下垃圾回收
> 
Java 垃圾回收机制最基本的做法是分代回收。内存中的区域被划分成不同的世代，对象根据其存活的时间被保存在对应世代的区域中。一般的实现是划分成3个世代：年轻、年老和永久。内存的分配是发生在年轻世代中的。当一个对象存活时间足够长的时候，它就会被复制到年老世代中。对于不同的世代可以使用不同的垃圾回收算法。进行世代划分的出发点是对应用中对象存活时间进行研究之后得出的统计规律。一般来说，一个应用中的大部分对象的存活时间都很短。比如局部变量的存活时间就只在方法的执行过程中。基于这一点，对于年轻世代的垃圾回收算法就可以很有针对性.

- 调用System.gc()会发生什么?
> 
通知GC开始工作,但是GC真正开始的时间不确定.

- 进程,线程之间的区别
> 
简而言之,进程是程序运行和资源分配的基本单位,一个程序至少有一个进程,一个进程至少有一个线程.进程在执行过程中拥有独立的内存单元,而多个线程共享内存资源,减少切换次数,从而效率更高.线程是进程的一个实体,是cpu调度和分派的基本单位,是比程序更小的能独立运行的基本单位.同一进程中的多个线程之间可以并发执行.

- 创建两种线程的方式?他们有什么区别?
>
通过实现java.lang.Runnable或者通过扩展java.lang.Thread类.相比扩展Thread,实现Runnable接口可能更优.原因有二:
>
Java不支持多继承.因此扩展Thread类就代表这个子类不能扩展其他类.而实现Runnable接口的类还可能扩展另一个类.  
类可能只要求可执行即可,因此继承整个Thread类的开销过大.

- Thread类中的start()和run()方法有什么区别?
> 
start()方法被用来启动新创建的线程，而且start()内部调用了run()方法，这和直接调用run()方法的效果不一样。当你调用run()方法的时候，只会是在原来的线程中调用，没有新的线程启动，start()方法才会启动新线程。

- Runnable和Callable的区别
> 
Runnable接口中的run()方法的返回值是void，它做的事情只是纯粹地去执行run()方法中的代码而已；Callable接口中的call()方法是有返回值的，是一个泛型，和Future、FutureTask配合可以用来获取异步执行的结果。 这其实是很有用的一个特性，因为多线程相比单线程更难、更复杂的一个重要原因就是因为多线程充满着未知性，某条线程是否执行了？某条线程执行了多久？某条线程执行的时候我们期望的数据是否已经赋值完毕？无法得知，我们能做的只是等待这条多线程的任务执行完毕而已。而Callable+Future/FutureTask却可以方便获取多线程运行的结果，可以在等待时间太长没获取到需要的数据的情况下取消该线程的任务

- 线程阻塞
>
阻塞指的是暂停一个线程的执行以等待某个条件发生（如某资源就绪），学过操作系统的同学对它一定已经很熟悉了。Java 提供了大量方法来支持阻塞，下面让我们逐一分析。
>
- sleep()允许 指定以毫秒为单位的一段时间作为参数，它使得线程在指定的时间内进入阻塞状态，不能得到CPU 时间，指定的时间一过，线程重新进入可执行状态。 典型地，sleep() 被用在等待某个资源就绪的情形：测试发现条件不满足后，让线程阻塞一段时间后重新测试，直到条件满足为止
- suspend() 和 resume()	两个方法配套使用，suspend()使得线程进入阻塞状态，并且不会自动恢复，必须其对应的resume() 被调用，才能使得线程重新进入可执行状态。典型地，suspend() 和 resume() 被用在等待另一个线程产生的结果的情形：测试发现结果还没有产生后，让线程阻塞，另一个线程产生了结果后，调用 resume() 使其恢复。
- yield() 使当前线程放弃当前已经分得的CPU 时间，但不使当前线程阻塞，即线程仍处于可执行状态，随时可能再次分得 CPU 时间。调用 yield() 的效果等价于调度程序认为该线程已执行了足够的时间从而转到另一个线程
- wait() 和 notify()	两个方法配套使用，wait() 使得线程进入阻塞状态，它有两种形式，一种允许 指定以毫秒为单位的一段时间作为参数，另一种没有参数，前者当对应的 notify() 被调用或者超出指定时间时线程重新进入可执行状态，后者则必须对应的 notify() 被调用.

- wait()与sleep()的区别
> 
- sleep()来自Thread类，和wait()来自Object类.调用sleep()方法的过程中，线程不会释放对象锁。而 调用 wait 方法线程会释放对象锁
- sleep()睡眠后不出让系统资源，wait让其他线程可以占用CPU
- sleep(milliseconds)需要指定一个睡眠时间，时间一到会自动唤醒.而wait()需要配合notify()或者notifyAll()使用

- ThreadLoal的作用是什么?

Java提供ThreadLocal类来支持线程局部变量，是一种实现线程安全的方式。
简单说ThreadLocal就是一种以空间换时间的做法在每个Thread里面维护了一个ThreadLocal.ThreadLocalMap把数据进行隔离，数据不共享，自然就没有线程安全方面的问题了.

- 生产者消费者问题实现  
> 
[http://huachao1001.github.io/article.html?QhSkxKKX]()
>
wait() / notify()方法实现:
```java
public class Test { 
    private static Integer count = 0; 
    private final Integer FULL = 5; 
    private static String lock = "lock"; 
 
    public static void main(String[] args) { 
        Test t = new Test(); 
        new Thread(t.new Producer()).start(); 
        new Thread(t.new Consumer()).start(); 
        new Thread(t.new Producer()).start(); 
        new Thread(t.new Consumer()).start(); 
    } 
 
    class Producer implements Runnable { 
        @Override 
        public void run() { 
            for (int i = 0; i < 5; i++) { 
                try { 
                    Thread.sleep(1000); 
                } catch (InterruptedException e1) {  
                    e1.printStackTrace(); 
                } 
                synchronized (lock) { 
                    while (count == FULL) { 
                        try { 
                            lock.wait(); 
                        } catch (InterruptedException e) {  
                            e.printStackTrace(); 
                        } 
                    } 
                    count++; 
                    System.out.println("生产者"+Thread.currentThread().getName() 
                            + "已生产完成，商品数量：" + count); 
                    lock.notifyAll(); 
                } 
            } 
        } 
    } 
 
    class Consumer implements Runnable { 
        @Override 
        public void run() { 
            for (int i = 0; i < 5; i++) { 
                try { 
                    Thread.sleep(1000); 
                } catch (InterruptedException e1) { 
                    e1.printStackTrace(); 
                } 
                synchronized (lock) { 
                    while (count == 0) { 
                        try { 
                            lock.wait(); 
                        } catch (InterruptedException e) { 
                            e.printStackTrace(); 
                        } 
                    } 
                    count--; 
                    System.out.println("消费者"+Thread.currentThread().getName() 
                            + "已消费，剩余商品数量：" + count); 
                    lock.notifyAll(); 
                } 
            } 
        } 
    } 
}
```

- 为什么要使用线程池
>
避免频繁地创建和销毁线程，达到线程对象的重用。另外，使用线程池还可以根据项目灵活地控制并发的数目。

- Java中的集合及其继承关系
![](https://camo.githubusercontent.com/209ef52cb0869c23c94c5c92844a2fff44b77cd1/687474703a2f2f696d672e626c6f672e6373646e2e6e65742f32303134313130353139333133333831323f77617465726d61726b2f322f746578742f6148523063446f764c324a736232637559334e6b626935755a5851765a4751344e6a51784e4441784d7a413d2f666f6e742f3561364c354c32542f666f6e7473697a652f3430302f66696c6c2f49304a42516b46434d413d3d2f646973736f6c76652f37302f677261766974792f43656e746572)

- ArrayList和LinkedList的区别?
>
最明显的区别是 ArrrayList底层的数据结构是数组，支持随机访问，而 LinkedList 的底层数据结构是双向循环链表，不支持随机访问。使用下标访问一个元素，ArrayList 的时间复杂度是 O(1)，而 LinkedList 是 O(n)。

- ArrayList和Array有什么区别?
>
Array可以容纳基本类型和对象，而ArrayList只能容纳对象。  
Array是指定大小的，而ArrayList大小是固定的

- 遍历ArrayList时如何正确移除一个元素
```java
public static void remove(ArrayList<String> list) {  
    Iterator<String> it = list.iterator();  
    while (it.hasNext()) {  
        String s = it.next();  
        if (s.equals("bb")) {  
            it.remove();  
        }  
    }  
}  
```

- Fail-Fast机制
> 快速失败”也就是fail-fast，它是Java集合的一种错误检测机制。当多个线程对集合进行结构上的改变的操作时，有可能会产生fail-fast机制。记住是有可能，而不是一定。例如：假设存在两个线程（线程1、线程2），线程1通过Iterator在遍历集合A中的元素，在某个时候线程2修改了集合A的结构（是结构上面的修改，而不是简单的修改集合元素的内容），那么这个时候程序就会抛出 ConcurrentModificationException 异常，从而产生fail-fast机制。

- Fail-fast和Fail-safe有什么区别
> 
> Java.util包中的所有集合类都被设计为fail->fast的，而java.util.concurrent中的集合类都为fail-safe的。当检测到正在遍历的集合的结构被改变时，Fail-fast迭代器抛出ConcurrentModificationException，而fail-safe迭代器从不抛出ConcurrentModificationException。

- SimpleDateFormat是线程安全的吗?
> 
DateFormat 的所有实现，包括 SimpleDateFormat 都不是线程安全的，因此你不应该在多线程序中使用，除非是在对外线程安全的环境中使用，如 将 SimpleDateFormat 限制在 ThreadLocal 中。如果你不这么做，在解析或者格式化日期的时候，可能会获取到一个不正确的结果。因此，从日期、时间处理的所有实践来说，我强力推荐 joda-time 库。

- 简单描述java异常体系
![](http://img.blog.csdn.net/20150107221255554?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZGQ4NjQxNDAxMzA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

- throw和throws的区别
> 
throw用于主动抛出java.lang.Throwable 类的一个实例化对象，意思是说你可以通过关键字 throw 抛出一个 Error 或者 一个Exception，如：throw new IllegalArgumentException(“size must be multiple of 2″), 而throws 的作用是作为方法声明和签名的一部分，方法被抛出相应的异常以便调用者能处理。Java 中，任何未处理的受检查异常强制在 throws 子句中声明。

- JVM特性
> 
平台无关性. Java语言的一个非常重要的特点就是与平台的无关性。而使用Java虚拟机是实现这一特点的关键。一般的高级语言如果要在不同的平台上运行，至少需要编译成不同的目标代码。而引入Java语言虚拟机后，Java语言在不同平台上运行时不需要重新编译。Java语言使用模式Java虚拟机屏蔽了与具体平台相关的信息，使得Java语言编译程序只需生成在Java虚拟机上运行的目标代码（字节码），就可以在多种平台上不加修改地运行。Java虚拟机在执行字节码时，把字节码解释成具体平台上的机器指令执行。

- 简述堆和栈的区别
> 
VM 中堆和栈属于不同的内存区域，使用目的也不同。栈常用于保存方法帧和局部变量，而对象总是在堆上分配。栈通常都比堆小，也不会在多个线程之间共享，而堆被整个 JVM 的所有线程共享。

- 简述JVM内存分配
> 
基本数据类型比变量和对象的引用都是在栈分配的
堆内存用来存放由new创建的对象和数组
类变量（static修饰的变量），程序在一加载的时候就在堆中为类变量分配内存，堆中的内存地址存放在栈中
实例变量：当你使用java关键字new的时候，系统在堆中开辟并不一定是连续的空间分配给变量，是根据零散的堆内存地址，通过哈希算法换算为一长串数字以表征这个变量在堆中的"物理位置”,实例变量的生命周期--当实例变量的引用丢失后，将被GC（垃圾回收器）列入可回收“名单”中，但并不是马上就释放堆中内存
局部变量: 由声明在某方法，或某代码段里（比如for循环），执行到它的时候在栈中开辟内存，当局部变量一但脱离作用域，内存立即释放  
>
>简单点：   
栈：存放局部变量和对象的引用。  
堆：对象的实例  
方法区：静态变量和常量字符串  

- XML解析的几种方式和特点
>
DOM,SAX,PULL三种解析方式:
>
- DOM:消耗内存：先把xml文档都读到内存中，然后再用DOM API来访问树形结构，并获取数据。这个写起来很简单，但是很消耗内存。要是数据过大，手机不够牛逼，可能手机直接死机
- SAX:解析效率高，占用内存少，基于事件驱动的：更加简单地说就是对文档进行顺序扫描，当扫描到文档(document)开始与结束、元素(element)开始与结束、文档(document)结束等地方时通知事件处理函数，由事件处理函数做相应动作，然后继续同样的扫描，直至文档结束。
- PULL:与 SAX 类似，也是基于事件驱动，我们可以调用它的next（）方法，来获取下一个解析事件（就是开始文档，结束文档，开始标签，结束标签），当处于某个元素时可以调用XmlPullParser的getAttributte()方法来获取属性的值，也可调用它的nextText()获取本节点的值。

- JDK 1.7特性
> 
然 JDK 1.7 不像 JDK 5 和 8 一样的大版本，但是，还是有很多新的特性，如 try-with-resource 语句，这样你在使用流或者资源的时候，就不需要手动关闭，Java 会自动关闭。Fork-Join 池某种程度上实现 Java 版的 Map-reduce。允许 Switch 中有 String 变量和文本。菱形操作符(<>)用于类型推断，不再需要在变量声明的右边申明泛型，因此可以写出可读写更强、更简洁的代码

- JDK 1.8特性
>
-  Lambda 表达式(也称为闭包)，允许我们将函数当成参数传递给某个方法，或者把代码本身当作数据处理.
-  接口的默认方法和静态方法,接口提供的默认方法会被接口的实现类继承或者覆写。
-  Stream API，充分利用现代多核 CPU，可以写出很简洁的代码 
-  Date 与 Time API，最终，有一个稳定、简单的日期和时间库可供你使用 扩展方法。
-  重复注解，现在你可以将相同的注解在同一类型上使用多次。