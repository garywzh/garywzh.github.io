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

