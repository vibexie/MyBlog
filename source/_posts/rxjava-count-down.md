---
title: RxJava实现倒计时
date: 2017-07-23 22:20:46
tags: rxjava
---
Just see the below 60s-count-down example:
```java
final int countdown = 60;
Observable.interval(1, TimeUnit.SECONDS)
        .take(countdown + 1)
        .flatMap(value -> Observable.just(countdown - Long.valueOf(value).intValue()))
        .subscribe(value -> System.out.println("count: " + value)
                , throwable -> System.out.println(throwable.getMessage())
                , () -> System.out.println("compete count down!"));
```
Output:
```shell
count: 60
count: 59
.
.
.
count: 2
count: 1
count: 0
compete count down!
```
Wow, very short code!