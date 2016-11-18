---
title: Java的集合与泛型
date: 2016-06-10 09:30:00
tags: Java
---

假设 `class Dog extends Animal`

### 泛型集合违反直觉的行为

因为Java有多态的特性，所以我们可以这样写:

`Animal animal = new Dog();`

在Java集合中类似的这样写就会提示错误

`List<Animal> list = new ArrayList<Dog>();`

A dog is a animal, but a list of dogs is not a list of animals. 这看起来很违反直觉，但Java为了类型安全就是这么规定的

原因如下:

Java规定 如果声明了 `animals` 是一个 `List<Animal>`, 那么一定允许执行 `animals.add(new Animal());`

但是我们知道 如果声明了 `dogs` 是一个 `List<Dog>`, 那么 `dogs.add(new Animal());` 是一定会报错的

所以Java规定了 一个 `List<Dog>` 不是一个 `List<Animal>`

### 泛型通配符

那么如果我们想把一个 `List<Dog>` 转换成一个 `List<Animal>` 要怎么做呢？我们可以定义一个方法

```java
public static void copy(List<Animal> animals, List<Dog> dogs) {
	for (Dog dog : dogs) {
		animals.add(dog);
	}
}
```

将原list中的对象逐个添加到新list中,这种对集合操作的方法我们可以把它抽象出来 对象类型之间的继承关系可以用泛型中的通配符来表示

`public static <T> void copy(List<T> dest, List<? extends T> src)`

或者

`public static <T> void copy(List<? super T> dest, List<T> src)`

或者

`public static <T> void copy(List<? super T> dest, List<? extends T> src)`

泛型中使用通配符有 [get-put principle](http://stackoverflow.com/a/1292147/5435312), 也就是如果使用了`<? extends T>`只能get不能put(可以put null),如果使用了`<? super T>`只能put不能get(只能get到 Object)

java.util.Collections中的`copy`使用的就是最后一种写法,更符合 [PECS](http://stackoverflow.com/a/2723538/5435312), 这在某种意义上限制了部分操作，减小了代码出错的几率

