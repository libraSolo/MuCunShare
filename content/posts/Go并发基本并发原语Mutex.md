# Go并发_基本并发原语_Mutex

# Mutex (互斥锁/排他锁)

![请输入图片描述](http://mucunliangtai.com/usr/uploads/2024/08/3145499981.jpg)

**临界区** ： 在并发编程中，如果程序中的一部分会被并发访问或修改，这部分程序需要被保护起来，这部分被保护起来的程序，就叫做临界区。

**适用场景** ：

1. 共享资源
2. 任务编排
3. 消息传递

**检测工具**：Go race detector (在运行时，触发才可以检测)

```go
$ go test -race mypkg    // test the package
$ go run -race mysrc.go  // compile and run the program
$ go build -race mycmd   // build the command
$ go install -race mypkg // install the package
```

## Mutex 的基本使用

```go
type Mutex struct {
 state int32
 sema uint32 
}
// state 字段被分为 4 部分
const (
	mutexLocked           = 1 << iota // 持有锁的标记
	mutexWoken                        // 唤醒标记
	mutexStarving                     // 饥饿标记
	mutexWaiterShift      = iota      // 阻塞等待的 waiter 数量,其最大为 2^(32-3)-1
)
```

方法：

```go
  func(m *Mutex)Lock() // 获得锁
  func(m *Mutex)Unlock() // 释放锁
```

当一个 goroutine 通过调用 Lock 方法获得了这个锁的拥有权后， 其它请求锁的 goroutine 就会阻塞在 Lock 方法的调用上，直到锁被释放并且自己获取到了这个锁的拥有权。

```go
var mu sync.Mutex // Mutex 的零值是还没有 goroutine 等待的未加锁的状态，所以你不需要额外的初始化，直接声明变量（如 var mu sync.Mutex）即可
```

### 注意

1. 初始化 strcut，不必初始化 Mutex 字段
2. 匿名嵌入 Mutex，可以直接调用其方法
3. 多个字段，把 Mutex 放在要控制的字段上面，便于维护。

### 锁获取机制

等待的 goroutine 们是以 FIFO 排队的

1. 当 Mutex 处于**正常模式**时，若此时没有新 goroutine 与队头 goroutine 竞争，则队头 goroutine 获得。若有新 goroutine 竞争大概率新 goroutine 获得。
2. 当队头 goroutine 竞争锁失败 1ms 后，它会将 Mutex 调整为**饥饿模式**。进入饥饿模式后，锁的所有权会直接从解锁 goroutine 移交给队头 goroutine ，此时新来的 goroutine 直接放入队尾。
3. 当一个 goroutine 获取锁后，如果发现自己满足下列条件中的任何一个：
   1. 它是队列中最后一个。
   2. 它等待锁的时间少于 1ms ，则将锁切换回正常模式。

## Mutex 错误场景

1. Lock/Unlock 不是成对出现
2. Copy 已使用的 Mutex
3. 重入
4. 死锁

### Lock/Unlock 不是成对出现

**策略**：善用 defer 或者用完就关

### Copy 已使用的 Mutex

**场景**：如果你要复制一个已经加锁的 Mutex 给一个新的变量，那么新的刚初始化的变量居然被加锁了

**原理**：mutex 的 state 是公共的，如果复制这个对象之前已经是加锁状态，那么赋值之后就已经有锁了。

```go
type Counter struct {
    sync.Mutex
    Count int
}


func main() {
    var c Counter
    c.Lock()
    defer c.Unlock()
    c.Count++
    foo(c) // 复制锁
}

// 这里Counter的参数是通过复制的方式传入的
func foo(c Counter) {
    c.Lock()
    defer c.Unlock()
    fmt.Println("in foo")
}
```

**策略**：使用 go vet 检查，`go vet xxx.go` 及时发现问题。使用了 Locker 接口的就会被分析到。

### 重入

**概念**：一个函数拥有某个锁后，还可以不断调用锁，叫可重入锁(递归锁)。

**注意： Mutex 不是可重入的锁。**

#### 实现可重入锁

1. 通过 hacker 的方式获取到 goroutine id，记录下获取锁的 goroutine id，它可以实现 Locker 接口。
2. 调用 Lock/Unlock 方法时，由 goroutine 提供一个 token，用来标识它自己。

##### 1.goroutine id

```go
package RecursiveMutex

import (
	"fmt"
	"github.com/petermattis/goid"
	"sync"
	"sync/atomic"
)

// RecursiveMutex 包装一个Mutex,实现可重入
type RecursiveMutex struct {
	sync.Mutex
	owner     int64 // 当前持有锁的goroutine id
	recursion int32 // 这个goroutine 重入的次数
}

func (m *RecursiveMutex) Lock() {
	gid := goid.Get()
	// 如果当前持有锁的goroutine就是这次调用的goroutine,说明是重入
	if atomic.LoadInt64(&m.owner) == gid {
		m.recursion++
		return
	}
	m.Mutex.Lock()
	// 获得锁的goroutine第一次调用，记录下它的goroutine id,调用次数加1
	atomic.StoreInt64(&m.owner, gid)
	m.recursion = 1
}

func (m *RecursiveMutex) Unlock() {
	gid := goid.Get()
	// 非持有锁的goroutine尝试释放锁，错误的使用
	if atomic.LoadInt64(&m.owner) != gid {
		panic(fmt.Sprintf("wrong the owner(%d): %d!", m.owner, gid))
	}
	// 调用次数减1
	m.recursion--
	if m.recursion != 0 { // 如果这个goroutine还没有完全释放，则直接返回
		return
	}
	// 此goroutine最后一次调用，需要释放锁
	atomic.StoreInt64(&m.owner, -1)
	m.Mutex.Unlock()
}

```

##### 2.token

```go
package RecursiveMutex

import (
	"fmt"
	"sync"
	"sync/atomic"
)

// Token方式的递归锁
type TokenRecursiveMutex struct {
	sync.Mutex
	token     int64
	recursion int32
}

// 请求锁，需要传入token
func (m *TokenRecursiveMutex) Lock(token int64) {
	if atomic.LoadInt64(&m.token) == token { //如果传入的token和持有锁的token一致，说明是递归调用
		m.recursion++
		return
	}
	m.Mutex.Lock() // 传入的token不一致，说明不是递归调用
	// 抢到锁之后记录这个token
	atomic.StoreInt64(&m.token, token)
	m.recursion = 1
}

// 释放锁
func (m *TokenRecursiveMutex) Unlock(token int64) {
	if atomic.LoadInt64(&m.token) != token { // 释放其它token持有的锁
		panic(fmt.Sprintf("wrong the owner(%d): %d!", m.token, token))
	}
	m.recursion-- // 当前持有这个锁的token释放锁
	if m.recursion != 0 { // 还没有回退到最初的递归调用
		return
	}
	atomic.StoreInt64(&m.token, 0) // 没有递归调用了，释放锁
	m.Mutex.Unlock()
}

```

### 死锁

**概念**：两个或两个以上的进程（或线程，goroutine）在执行过程中，因争夺共享资源而处于一种互相等待的状态，如果没有外部干涉，它们都将无法推进下去，此时，我们称系统处于死锁状态或系统产生了死锁。

**必要条件**

1. 互斥
2. 持有和等待
3. 不可剥夺
4. 环路等待

## Mutex 扩展

### TryLock

**概念**： 一个 goroutine 调用 TryLock， 能拿到就拿，拿不到直接返回 false 并不会阻塞。

### 获取等待者数量等指标

```go
package main

import (
	"fmt"
	"sync"
	"sync/atomic"
	"time"
	"unsafe"
)

const (
	mutexLocked = 1 << iota // mutex is locked
	mutexWoken
	mutexStarving
	mutexWaiterShift = iota
)

type Mutex struct {
	sync.Mutex
}

func (m *Mutex) Count() int {
	// 获取state字段的值
	v := atomic.LoadInt32((*int32)(unsafe.Pointer(&m.Mutex)))
	v = v>>mutexWaiterShift + (v & mutexLocked)
	return int(v)
}

// 锁是否被持有
func (m *Mutex) IsLocked() bool {
	state := atomic.LoadInt32((*int32)(unsafe.Pointer(&m.Mutex)))
	return state&mutexLocked == mutexLocked
}

// 是否有等待者被唤醒
func (m *Mutex) IsWoken() bool {
	state := atomic.LoadInt32((*int32)(unsafe.Pointer(&m.Mutex)))
	return state&mutexWoken == mutexWoken
}

// 锁是否处于饥饿状态
func (m *Mutex) IsStarving() bool {
	state := atomic.LoadInt32((*int32)(unsafe.Pointer(&m.Mutex)))
	return state&mutexStarving == mutexStarving
}

func count() {
	var mu Mutex
	for i := 0; i < 1000; i++ { // 启动1000个goroutine
		go func() {
			mu.Lock()
			time.Sleep(time.Second)
			mu.Unlock()
		}()
	}

	time.Sleep(time.Second)
	// 输出锁的信息
	fmt.Printf("waitings: %d, isLocked: %t, woken: %t,  starving: %t\n", mu.Count(), mu.IsLocked(), mu.IsWoken(), mu.IsStarving())
}

func main() {
	count()
}