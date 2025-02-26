+++
date = '2024-8-24T16:21:57+08:00'
draft = false
title = 'Go并发_原子操作_atomic'
author = '木村凉太'
categories = 'Golang'
hiddenFromHomePage = true 
+++

# Go并发_原子操作_atomic

# 原子操作

![请输入图片描述](http://mucunliangtai.com/usr/uploads/2024/08/393808069.jpg)

原子操作是指在并发编程中，某个操作要么完全执行，要么完全不执行，不会被其他线程或任务打断。这种特性确保了在多线程环境下的操作的完整性和一致性。

## 方法

### Add

`func AddInt32(addr *int32, delta int32) (new int32)`

可以加一个负数， 对于无符号的整数来说，可以利用补码规则，由减法变加法。

### CAS（CompareAndSwap）

`func CompareAndSwapInt32(addr *int32, old, new int32) (swapped bool)`

比较当前 addr 地址里的值是不是 old，如果不等于 old，就返回 false；如果等于 old，就把此地址的值替换成 new 值，返回 true。这就相当于“判断相等才替换”。

### Swap

`func SwapInt32(addr *int32, new int32) (old int32)`

不需要比较相等，直接替换。

### Load

Load 方法会取出 addr 地址中的值，即使在多处理器、多核、有 CPU cache 的情况下，这个操作也能保证 Load 是一个原子操作。

### Store

Store 方法会把一个值存入到指定的 addr 地址中，即使在多处理器、多核、有 CPU cache 的情况下，这个操作也能保证 Store 是一个原子操作。别的 goroutine 通过 Load 读取出来，不会看到存取了一半的值。

### Value

stomic 的特殊类型，Value。它可以CAS/Swap/Load/Store。

测试用法：

```go
package main

import (
	"log"
	"math/rand"
	"sync"
	"sync/atomic"
	"time"
)

type Config struct {
	NodeName string
	Addr     string
	Count    int32
}

func loadNewConfig() Config {
	return Config{
		NodeName: "1",
		Addr:     "2",
		Count:    rand.Int31(),
	}
}

func main() {
	var config atomic.Value
	config.Store(loadNewConfig())
	cond := sync.NewCond(&sync.Mutex{})

	go func() {
		for {
			time.Sleep(time.Second * 3)
			config.Store(loadNewConfig())
			cond.Broadcast()
		}
	}()
	go func() {
		for {
			cond.L.Lock()
			cond.Wait()
			c := config.Load().(Config)
			log.Printf("%+v", c)
			cond.L.Unlock()
		}
	}()

	select {}

}