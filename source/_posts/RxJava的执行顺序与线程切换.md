---
title: RxJava的执行顺序与线程切换
date: 2016-06-28 11:40:28
tags: RxJava
---

RxJava的操作可以分为两类:准备阶段，执行阶段。

准备阶段的操作有：
subscriber.onStart()
doOnSubscribe()
subscribeOn()

执行阶段的操作有：
doOnNext()
各种操作符(map(), flatMap(), filter(), ..)
observeOn()
subscriber.onNext()
subscriber.onComplete()
subscriber.onError()

需要了解的是：不管各个操作的书写先后顺序，所有的准备操作都会先挑出来运行，然后才去运行剩下的执行操作。所以说，准备阶段操作和执行阶段的操作在代码书写顺序上可以随便交换。如下,两类操作用了不同的缩进以示区别：

```java
Observable.just()3
			.doOnNext()4
	.doOnSubscribe()2
			.map()5
	.subscribeOn()1
			.observeOn()6
			.subscribe();7
```

```java
Observable.just()3
	.doOnSubscribe()2
	.subscribeOn()1
			.doOnNext()4
			.map()5
			.observeOn()6
			.subscribe();7
```

两段代码在程序执行上没有任何区别。但是为了增加代码的易读性，推荐写成第二种，把准备阶段的操作都连到一起写到前面，执行阶段的操作连在一起写到后面。

虽然不同类的操作（准备和执行两类）书写顺序可以随意交换不影响程序运行，但是同属于一类的操作的相对顺序在书写上有着严格的要求，因为程序的执行在准备阶段是从后往前（倒序），在执行阶段是从前往后（顺序），上图的序号代表了执行顺序。

下面是详细例子:

```java
package com.example;

import rx.Observable;
import rx.Subscriber;
import rx.functions.Action0;
import rx.functions.Action1;
import rx.functions.Func1;
import rx.schedulers.Schedulers;

public class TestClass {

    public static void main(String[] args) throws InterruptedException {

        Observable
                .create(new Observable.OnSubscribe<String>() {
                    @Override
                    public void call(Subscriber<? super String> subscriber) {
                        System.out.println("3_creat: " + Thread.currentThread());
                        subscriber.onNext("hello");
                        subscriber.onCompleted();
                    }
                })
                .doOnSubscribe(new Action0() {
                    @Override
                    public void call() {
                        System.out.println("2_subscribe: " + Thread.currentThread());
                    }
                })
                .subscribeOn(Schedulers.io())
                .doOnSubscribe(new Action0() {
                    @Override
                    public void call() {
                        System.out.println("1_subscribe: " + Thread.currentThread());
                        System.out.println("---changed to IO thread");
                    }
                })
                //分界线----------上面倒序执行，下面顺序执行（subscriber.onStart()特例最先执行）
                .doOnNext(new Action1<String>() {
                    @Override
                    public void call(String s) {
                        System.out.println("4_next: " + Thread.currentThread());
                    }
                })
                .map(new Func1<String, Integer>() {
                    @Override
                    public Integer call(String s) {
                        System.out.println("5_map: " + Thread.currentThread());
                        return s.length();
                    }
                })
                .doOnNext(new Action1<Integer>() {
                    @Override
                    public void call(Integer i) {
                        System.out.println("6_next: " + Thread.currentThread());
                        System.out.println("---changed to new thread");
                    }
                })
                .observeOn(Schedulers.newThread())
                .doOnNext(new Action1<Integer>() {
                    @Override
                    public void call(Integer i) {
                        System.out.println("7_next: " + Thread.currentThread());
                    }
                })
                .subscribe(new Subscriber<Integer>() {

                    @Override
                    public void onStart() {
                        System.out.println("0_start: " + Thread.currentThread());
                        super.onStart();
                    }

                    @Override
                    public void onCompleted() {

                    }

                    @Override
                    public void onError(Throwable e) {

                    }

                    @Override
                    public void onNext(Integer integer) {
                        System.out.println("8_next: " + Thread.currentThread());
                    }
                });

        Thread.sleep(2000);//等待其他线程结束
    }
}
```

代码会按序号顺序执行,输出为:

```
0_start: Thread[main,5,main]
1_subscribe: Thread[main,5,main]
---changed to Io thread
2_subscribe: Thread[RxIoScheduler-2,5,main]
3_creat: Thread[RxIoScheduler-2,5,main]
4_next: Thread[RxIoScheduler-2,5,main]
5_map: Thread[RxIoScheduler-2,5,main]
6_next: Thread[RxIoScheduler-2,5,main]
---changed to new thread
7_next: Thread[RxNewThreadScheduler-1,5,main]
8_next: Thread[RxNewThreadScheduler-1,5,main]
```

