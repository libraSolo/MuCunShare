+++
date = '2024-8-24T16:21:57+08:00'
draft = false
title = 'Go并发_基本并发原语_Pool'
author = '木村凉太'
categories = 'Golang'
hiddenFromHomePage = true 
+++

# Go并发_基本并发原语_Pool

# Pool

![请输入图片描述](http://mucunliangtai.com/usr/uploads/2024/08/2059974212.jpg)

sync.Pool 用与缓存**临时**使用的对象，避免反复创建带来的性能损耗。所以如果没有别的对象引用，会被垃圾回收掉。

## 特性

1. sync.Pool 本身就是线程安全的，多个 goroutine 可以并发地调用它的方法存取对象
2. sync.Pool 不可在使用之后再复制使用。

## 方法

New、Get 和 Put。

1. New
   Pool struct 包含一个 New 字段，这个字段的类型是函数 func() interface{}。当调用 Pool 的 Get 方法从池中获取元素，没有更多的空闲元素可返回时，就会调用这个 New 方法来创建新的元素。如果你没有设置 New 字段，没有更多的空闲元素可返回时，Get 方法将返回 nil，表明当前没有可用的元素。
2. Get
   从 Pool 取走一个元素，从 Pool 移除并返回。可能为 nil（Pool.New 字段没有设置，又没有空闲元素可以返回）。
3. Put
   将一个元素提交个 Pool， 提交一个 nil 值，直接忽略。

## 原理

```go
type Pool struct {
	noCopy noCopy

	local     unsafe.Pointer // local fixed-size per-P pool, actual type is [P]poolLocal
	localSize uintptr        // size of the local array

	victim     unsafe.Pointer // local from previous cycle
	victimSize uintptr        // size of victims array

	New func() any
}
```

在 STW 时，victim 被清空，local 赋值给 victim，local 置为 nil，Get 取值的时候是从 victim 中获取。

### local 字段

local 字段包含一个 poolLocalInternal 字段，并提供 CPU 缓存对齐，从而避免 false sharing。

poolLocalInternal 也包含两个字段：private 和 shared。

* private，代表一个缓存的元素，而且只能由相应的一个 P 存取。因为一个 P 同时只能执行一个 goroutine，所以不会有并发的问题。
* shared，可以由任意的 P 访问，但是只有本地的 P 才能 pushHead/popHead，其它 P 可以 popTail，相当于只有一个本地的 P 作为生产者（Producer），多个 P 作为消费者（Consumer），它是使用一个 local-free 的 queue 列表实现的。