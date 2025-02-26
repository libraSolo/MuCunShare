+++
date = '2024-8-15T16:21:57+08:00'
draft = false
title = 'Go并发_基本并发原语_WaitGroup'
author = '木村凉太'
categories = 'Golang'
hiddenFromHomePage = true 
+++

# Go并发_基本并发原语_WaitGroup

# WaitGroup (协同等待，任务编排利器)

## 方法

```go
func (wg *WaitGroup) Add(delta int) // 用来设置 WaitGroup 的计数值
func (wg *WaitGroup) Done()         // 用来将 WaitGroup 的计数值减 1，其实就是调用了 Add(-1)
func (wg *WaitGroup) Wait()         // 调用这个方法的 goroutine 会一直阻塞，直到 WaitGroup 的计数值变为 0
```

## 基本使用

```go
type WaitGroup struct {
    // 避免复制使用的一个技巧，可以告诉vet工具违反了复制使用的规则
    noCopy noCopy
    // 64bit(8bytes)的值分成两段，高32bit是计数值，低32bit是waiter的计数
    // 另外32bit是用作信号量的
    // 因为64bit值的原子操作需要64bit对齐，但是32bit编译器不支持，所以数组中的元素在不同的架构中不一样，具体处理看下面的方法
    // 总之，会找到对齐的那64bit作为state，其余的32bit做信号量
    state1 [3]uint32
}


// 得到state的地址和信号量的地址
func (wg *WaitGroup) state() (statep *uint64, semap *uint32) {
    if uintptr(unsafe.Pointer(&wg.state1))%8 == 0 {
        // 如果地址是64bit对齐的，数组前两个元素做state，后一个元素做信号量
        return (*uint64)(unsafe.Pointer(&wg.state1)), &wg.state1[2]
    } else {
        // 如果地址是32bit对齐的，数组后两个元素用来做state，它可以用来做64bit的原子操作，第一个元素32bit用来做信号量
        return (*uint64)(unsafe.Pointer(&wg.state1[1])), &wg.state1[0]
    }
}
```

64 位环境下：waiter 数 | WaitGroup 的计数值 | 信号量
32 位环境下：信号量 | waiter 数 | WaitGroup 的计数值

## 常见问题

### 1.计数器设置为负值

WaitGroup 的计数器的值必须大于等于 0，否则会 panic。

虽然 Add 可以传入负值，但是还是要用 Done 比较稳妥。

### 2. 调用 Done 方法的次数过多，超过了 WaitGroup 的计数值

设置多少次，调用多少次的 Done。否则会造成死锁(Done 次数调用少于设置值)，或者造成 Panic (Done 次数调用多于设置值)。

### 3. 不期望的 Add 时机

等所有的 Add 方法调用之后再次调用 Wait。

### 4. 前一个 Wait 还没结束就重用 WaitGroup

只要 WaitGroup 的值重归于 0 就可以重新使用，否则会 panic。

## noCopy: 辅助 vet 检查

它就是指示 vet 工具在做检查的时候，这个数据结构不能做值复制使用。更严谨地说，是不能在第一次使用之后复制使用。