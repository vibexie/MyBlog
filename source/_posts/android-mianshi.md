---
title: 安卓面试题记录
date: 2016-11-30 08:59:47
tags: 面试
---

#### Java基础
* String 类可以被继承吗？为什么？
答：不可以继承，String，Integer，Float，Double等包装类都不可以被继承，因为Java给他们定义为了final。

* Java equals()方法能被重写吗？
答：可以被重写。
判断两个对象在逻辑上是否相等，如根据类的成员变量来判断两个类的实例是否相等，而继承Object中的equals方法只能判断两个引用变量是否是同一个对象。这样我们往往需要重写equals()方法。
我们向一个没有重复对象的集合中添加元素时，集合中存放的往往是对象，我们需要先判断集合中是否存在已知对象，这样就必须重写equals方法。
重写equals方法的要求：
1.自反性：对于任何非空引用x，x.equals(x)应该返回true。
2.对称性：对于任何引用x和y，如果x.equals(y)返回true，那么y.equals(x)也应该返回true。
3.传递性：对于任何引用x、y和z，如果x.equals(y)返回true，y.equals(z)返回true，那么x.equals(z)也应该返回true。
4.一致性：如果x和y引用的对象没有发生变化，那么反复调用x.equals(y)应该返回同样的结果。
5.非空性：对于任意非空引用x，x.equals(null)应该返回false。
[详解](http://huaxia524151.iteye.com/blog/729074)
<!-- more -->

* 单例模式你会怎么写？
对于单例模式，我们需要兼顾线程安全和效率。
```
public class Singleton {
    private static volatile Singleton singleton = null;
     
    private Singleton(){}
     
    public static Singleton getSingleton(){
        if(singleton == null){
            synchronized (Singleton.class){
                if(singleton == null){
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }    
}
```

* 解释一下volatile关键字
在当前的Java内存模型下，线程可以把变量保存在本地内存（比如机器的寄存器）中，而不是直接在主存中进行读写。这就可能造成一个线程在主存中修改了一个变量的值，而另外一个线程还继续使用它在寄存器中的变量值的拷贝，造成数据的不一致。
要解决这个问题,把该变量声明为volatile（不稳定的）即可，这就指示JVM，这个变量是不稳定的，每次使用它都到主存中进行读取。一般说来，多任务环境下各任务间共享的标志都应该加volatile修饰。 

* Java中&&与&，||与|的区别？
&& 表示第一个条件满足时，不再判断后面的条件
& 表示第一个条件满足时，会继续判断后面的条件
|| 表示第一个条件满足时，不再判断后面的条件
| 表示第一个条件满足时，继续判断后面的条件
特别注意不能弄反了，记住&&和||才是常用的。

* 对于finally
即使在try中return了，下列代码依旧将会执行到finally的代码块中，所以我们针对java或者其他语言来说，我们要肯定它的语义和规范。
```java
public static void test () {
	try {
		return;
	} catch (Exception e) {
		System.out.println("catch the exception");
	} finally {
		System.out.println("finally");
	}
}
```

* 关于String的两种赋值方式
直接赋值的话，String s = "hello"会首先去strings缓存池（java源码注释描述为A pool of strings）中寻找hello，如果有，则直接复用，如同java基本数据类型。
```java
public void test () {
	String s = "hello";
	if (s == "hello") {
		System.out.println("hello");
	}
	
	//将输出hello
}
```
而通过new String("hello")的方式创建对象，每次都是从堆中new出新对象。
```java
public void test () {
    String s2 = new String("hello");
	String s = "hello";
	if (s == s2) {
		System.out.println("hello");
	}
	
	//将不会输出hello
}
```
我们不要认为和之前所说的，s寻找到s2这个对象已经存在，就复用了s2，所以s和s2应该相等，其实不是这样，strings缓存池和堆不是同一个概念，但是非要这样的话，需要将s2通过intern()方法先入池：
```java
public void test () {
	String s2 = new String("hello").intern();
	String s = "hello";
	if (s == s2) {
		System.out.println("hello");
	}
	//将会输出hello
}
```

#### Java后台
* 用过哪些消息队列MQ？消息队列的使用场景是怎样的？
答：用过RabbitMQ。
1.异步处理：用户注册后，需要发注册邮件和注册短信。传统的做法有两种1.串行的方式（如先发邮件后发短信，按顺序来）；2.并行方式（同时发邮件和短信）。
2.应用解耦
3.流量削锋
4.日志处理
5.消息通讯
[详解](http://www.cnblogs.com/stopfalling/p/5375492.html)

#### Android
* 如何加快应用启动速度？
安卓应用的启动方式分为热启动和冷启动。简单来说冷启动就是完完全全从零开始启动一个App，热启动是在之前已有的这个App的进程中重新启动。当然还有一种情况就是我们通过Home键将App移到后台，再进入App时那就叫恢复了。
App的启动时间是从点击应用图标到看到App第一帧界面（空白界面不算）的时间。通常使用下面命令测量
``` shell
adb shell am start -W [packageName]/[packageName.MainActivity]
```
加快启动速度方式：
1.在Application的构造器方法、attachBaseContext()、onCreate()方法中不要进行耗时操作的初始化，一些数据预取放在异步线程中，可以采取Callable实现。
2.对于sp的初始化，因为sp的特性在初始化时候会对数据全部读出来存在内存中，所以这个初始化放在主线程中不合适，反而会延迟应用的启动速度，对于这个还是需要放在异步线程中处理。
3.对于MainActivity，由于在获取到第一帧前，需要对contentView进行测量布局绘制操作，尽量减少布局的层次，考虑StubView的延迟加载策略，当然在onCreate、onStart、onResume方法中避免做耗时操作。
4.在等待第一帧显示的时间里，我们可以在Manifest中的MainActivity设置自定义的android:theme背景，然后再MainActivity中的onCreate()方法最前面这种默认theme即setTheme(R.style.AppTheme);这样的话，在MainActivity没有绘制完成之前，是显示自定义的背景，绘制完成之后，则显示AppTheme背景。这样极大提高了用户体验。

* Android OOM出现的原因和解决方案？
我们首先得搞清楚内存泄露和内存溢出的区别。
OOM即OutOfMemory，也就是堆内存溢出，大多数安卓机器的堆内存为16M（少数为24M）。
Memory leak是申请内存后，无法释放已申请的内存空间。
可以明确的是内存泄露最终导致内存溢出。

虽然Java有垃圾回收机制，但也存在内存泄露。如果我们一个程序中，已经不再使用某个对象，但是因为仍然有引用指向它，垃圾回收器就无法回收它，当然 该对象占用的内存就无法被使用，这就造成了内存泄露。
一般而言，android中常见的原因主要有以下几个：
1.随意使用static关键字。我们在使用static的时候，需要谨慎的考虑是否会导致OOM。
2.Context泄漏。Context尽量使用Application Context，因为Application的Context的生命周期比较长，引用它不会出现内存泄露的问题。
3.Activity泄露。如果Activity中有一个线程内部类，内部类将持有Activity的一个引用，当需要重新创建Activity的时候，如果旧Activity实例中有线程还没有结束，那么旧Activity将不会被回收，导致OOM。解决方案就是如果必须在Activity中创建线程内部类，最好使用静态内部类。
4.未关闭的IO流。
5.数据库的cursor没有关闭。
6.调用registerReceiver()后未调用unregisterReceiver()。
7.ListView没有adapter没有使用缓存contentview。
8.Bitmap内存泄露。绝大多数导致OOM的原因都是bitmap内存泄露。我们可以通过下列方式解决Bitmap内存泄露问题：
-a:及时的使用recycle方法回收bitmap，同时将引用置为null。
-b:捕获OutOfMemoryError异常而不是直接捕获Exception，发生了OutOfMemoryError异常后可以设置默认bitmap。
-c:使用缓存。借鉴Fresco使用的三级缓存方案，Bitmap缓存;未解码图片的内存缓存;文件缓存;当然，最后再通过网络缓存。
-d:压缩图片。
-e:终极方案，使用第三方图片框架。

* Android数据存储方式
1.使用SharedPreferences存储数据
2.文件存储数据      
3.SQLite数据库存储数据
4.使用ContentProvider存储数据
5.网络存储数据
注意ContentProvider这种存储方式不能忘！

#### 算法
* 输入两个整数n和m,从数列1,2,3，……n中随意取几个数，使其和等于m。要求将所有的可能组合列出来
求解思路：
1.首先判断，如果n>m，则n中大于m的数不可能参与组合，此时置n = m;
2.将最大数n加入且n == m,则满足条件，输出；
3.将n分两种情况求解，（1）n没有加入，取n = n - 1; m = m;递归下去；（2）n加入，取n = n - 1l, m = m - n,递归下去


