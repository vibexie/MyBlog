---
title: RxJava操作符之错误处理
date: 2017-02-24 21:51:07
tags: rxjava
---
#### onErrorReturn
![onErrorReturn](http://qiniu.vibexie.com/blog/rxjava-error-handle-1.png-width500)

<!-- more -->
在抛出异常的时候，onErrorReturn会拦截住异常，并且发出一个自己定义的同类型的消息继续执行，并且结束Observale。
在下面示例中，当消息3抛出异常时，被onErrorReturn拦截，onErrorReturn同时发射消息10，待消息10处理完后结束Observale。

示例
``` java
public void test() {
	Observable.just(1, 2, 3, 4, 5).doOnNext(value -> {
		if (value == 3) {
			int i = 1 / 0;
		}
	})
	.onErrorReturn(throwable -> 10).subscribe(
			new Observer<Integer>() {
				@Override
				public void onCompleted() {
					// TODO Auto-generated method stub
					System.out.println("onCompleted");
				}
				
				@Override
				public void onNext(Integer arg0) {
					// TODO Auto-generated method stub
					System.out.println("onNext " + arg0);
					
				}
				
				@Override
				public void onError(Throwable arg0) {
					// TODO Auto-generated method stub
					System.out.println("onError " + arg0.getMessage());
				}
			});
}
```
运行结果：
``` shell
onNext 1
onNext 2
onNext 100
onNext 101
onNext 102
onCompleted
```
