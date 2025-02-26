---
date : 2024-08-14T16:21:57+08:00
draft : false
title : 'Go并发_基本并发原语_RWMutex'
author : '木村凉太'
categories : ['Golang']
hiddenFromHomePage : true 
---

# Go并发_基本并发原语_RWMutex

# RWMutex

![请输入图片描述](http://mucunliangtai.com/usr/uploads/2024/08/2090840451.jpg)

**概念**：RWMutex 在某一时刻只能由任意数量的 reader 持有，或者是只被单个的 writer 持有。

**方法**：

* **Lock/Unlock**: 如果锁已经被 reader 或者 writer 持有，那么，Lock 方法会一直阻塞，直到能获取到锁；Unlock 则是配对的释放锁的方法。
* **RLock/Unlock**：如果锁已经被 writer 持有的话，RLock 方法会一直阻塞，直到能获取到锁，否则就直接返回；而 RUnlock 是 reader 释放锁的方法。
* **Rlocker**：这个方法的作用是为读操作返回一个 Locker 接口的对象。它的 Lock 方法会调用 RWMutex 的 RLock 方法，它的 Unlock 方法会调用 RWMutex 的 RUnlock 方法。
* **TryRLock/TryLock**: 尝试获取，同 Mutex。

其他用法和注意和 Mutex 相似。

## 常见问题

1. **Read-preferring**：读优先的设计可以提供很高的并发性，但是，在竞争激烈的情况下可能会导致写饥饿。
2. **Write-preferring**：避免写饥饿，阻塞的是新来 reader， 让已经在工作的 reader 工作完成。
3. **不指定优先级**：平等对待，解决饥饿。

**Go 标准库中的 RWMutex 设计是 Write-preferring 方案。一个正在阻塞的 Lock 调用会排除新的 reader 请求到锁。**

## 基本使用

```go
type RWMutex struct {
  w           Mutex   // 互斥锁解决多个writer的竞争
  writerSem   uint32  // writer信号量
  readerSem   uint32  // reader信号量
  readerCount int32   // reader的数量  记录当前 reader 的数量（以及是否有 writer 竞争锁）
  readerWait  int32   // 记录 writer 请求锁时需要等待 read 完成的 reader 的数量
}
```

### RLock/RUnLock

```go
func (rw *RWMutex) RLock() {
    if atomic.AddInt32(&rw.readerCount, 1) < 0 {
            // rw.readerCount是负值的时候，意味着此时有writer等待请求锁，因为writer优先级高，所以把后来的reader阻塞休眠
        runtime_SemacquireMutex(&rw.readerSem, false, 0)
    }
}
func (rw *RWMutex) RUnlock() {
    if r := atomic.AddInt32(&rw.readerCount, -1); r < 0 {
        rw.rUnlockSlow(r) // 有等待的writer
    }
}
func (rw *RWMutex) rUnlockSlow(r int32) {
    if atomic.AddInt32(&rw.readerWait, -1) == 0 {
        // 最后一个reader了，writer终于有机会获得锁了
        runtime_Semrelease(&rw.writerSem, false, 1)
    }
}
```

### Lock

一旦一个 writer 获得了内部的互斥锁，就会反转 readerCount 字段，把它从原来的正整数 readerCount(>=0) 修改为负数（readerCount-rwmutexMaxReaders），让这个字段保持两个含义（既保存了 reader 的数量，又表示当前有 writer）。

```go
func (rw *RWMutex) Lock() {
    // 首先解决其他writer竞争问题
    rw.w.Lock()
    // 反转readerCount，告诉reader有writer竞争锁
    r := atomic.AddInt32(&rw.readerCount, -rwmutexMaxReaders) + rwmutexMaxReaders
    // 如果当前有reader持有锁，那么需要等待
    if r != 0 && atomic.AddInt32(&rw.readerWait, r) != 0 {
        runtime_SemacquireMutex(&rw.writerSem, false, 0)
    }
}
```

### Unlock

当一个 writer 释放锁的时候，它会再次反转 readerCount 字段，即加上 rwmutexMaxReaders 这个常数，变成了正数。

```go
func (rw *RWMutex) Unlock() {
    // 告诉reader没有活跃的writer了
    r := atomic.AddInt32(&rw.readerCount, rwmutexMaxReaders)
  
    // 唤醒阻塞的reader们
    for i := 0; i < int(r); i++ {
        runtime_Semrelease(&rw.readerSem, false, 0)
    }
    // 释放内部的互斥锁
    rw.w.Unlock()
}
```

## 常见踩坑点

### 1. 不可复制

同 Mutex

### 2. 重入导致死锁

1. Mutex 本身重入就有问题，更别说基于它实现的读写锁，writer 重入调用 Lock  的时候，就会出现死锁。
2. reader 获取锁， writer 会阻塞等待，reader 重入时调用 writer 的写操作，互相依赖形成死锁。
3. 当一个 writer 请求锁的时候，如果已经有一些活跃的 reader，它会等待这些活跃的 reader 完成，如果之后活跃的 reader 再依赖新的 reader 的话，这些新的 reader 就会等待 writer 释放锁之后才能继续执行，这就形成了一个环形依赖：
   writer 依赖活跃的 reader -> 活跃的 reader 依赖新来的 reader -> 新来的 reader 依赖 writer。

第三种情况举例：

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	var mu sync.RWMutex

	// writer,稍微等待，然后制造一个调用Lock的场景
	go func() {
		time.Sleep(200 * time.Millisecond)
		mu.Lock()
		fmt.Println("Lock")
		time.Sleep(100 * time.Millisecond)
		mu.Unlock()
		fmt.Println("Unlock")
	}()

	go func() {
		factorial(&mu, 10) // 计算10的阶乘, 10!
	}()

	select {}
}

// 递归调用计算阶乘
func factorial(m *sync.RWMutex, n int) int {
	if n < 1 { // 阶乘退出条件
		return 0
	}
	fmt.Println("RLock")
	m.RLock()
	defer func() {
		fmt.Println("RUnlock")
		m.RUnlock()
	}()
	time.Sleep(100 * time.Millisecond)
	return factorial(m, n-1) * n // 递归调用
}
```

### 3.释放未加锁的 RWMutex

必须要成对出现 Lock/UnLock，RLock/RUnLock。