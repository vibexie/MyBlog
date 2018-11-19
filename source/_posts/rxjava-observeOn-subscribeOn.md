---
title: RxJava操作符之observeOn与subscribeOn
date: 2017-03-01 00:50:48
tags: rxjava
---
线程变换是Rxjava的重要特性，observeOn和subscribeOn两个方法承担了线程变换的重任。但是你真的会用observeOn和subscribeOn吗？

先看observeOn：
``` java
public void test() {
	Observable.just(1)
		.doOnNext(value -> System.out.println("rxjava step1:" + Thread.currentThread().getName()))
		.observeOn(Schedulers.newThread())
		.doOnNext(value -> System.out.println("rxjava step2:" + Thread.currentThread().getName()))
		.observeOn(Schedulers.io())
		.doOnNext(value -> System.out.println("rxjava step3:" + Thread.currentThread().getName()))
		.observeOn(Schedulers.newThread())
		.doOnNext(value -> System.out.println("rxjava step4:" + Thread.currentThread().getName()))
		.subscribe(value -> {
 			System.out.println("rxjava next :" + Thread.currentThread().getName());
		}, throwable -> {
			System.out.println("rxjava error:" + throwable.getMessage());
		});
}
```
<!-- more -->
``` shell
rxjava step1:main
rxjava step2:RxNewThreadScheduler-2
rxjava step3:RxIoScheduler-2
rxjava step4:RxNewThreadScheduler-1
rxjava next :RxNewThreadScheduler-1
```
可以看出，observeOn作用于该操作符之后直到出现新的observeOn操作符之间的作用域。

再看doOnSubscribe：
```java
public void test() {
	Observable.just(1)
		.doOnSubscribe(() -> System.out.println("rxjava step1:" + Thread.currentThread().getName()))
		.subscribeOn(Schedulers.newThread())
		.doOnSubscribe(() -> System.out.println("rxjava step2:" + Thread.currentThread().getName()))
		.subscribeOn(Schedulers.io())
        .doOnSubscribe(() -> System.out.println("rxjava step3:" + Thread.currentThread().getName()))
		.subscribeOn(Schedulers.newThread())
		.doOnSubscribe(() -> System.out.println("rxjava step4:" + Thread.currentThread().getName()))
		.subscribe(value -> {
        	System.out.println("rxjava next :" + Thread.currentThread().getName());
		}, throwable -> {
        	System.out.println("rxjava error:" + throwable.getMessage());
		});
}
```	

执行结果却是
``` shell
rxjava step4:main
rxjava step3:RxNewThreadScheduler-1
rxjava step2:RxIoScheduler-2
rxjava step1:RxNewThreadScheduler-2
rxjava next :RxNewThreadScheduler-2
```

也就是说，subscribeOn 作用于该操作符之前的 Observable 的创建操作符create（以上代码没有用create，简单用了just）以及 doOnSubscribe 操作符，换句话说就是 doOnSubscribe 以及 Observable 的创建操作符总是被其之后最近的 subscribeOn 控制。