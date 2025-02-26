# Go并发_基本并发原语_Cond


# Go并发_基本并发原语_Cond

# Cond

![请输入图片描述](http://mucunliangtai.com/usr/uploads/2024/08/3356714874.jpg)

**目的**：为等待 / 通知场景下的并发问题提供支持。

## 基本用法

Cond 并发原语初始化的时候，需要关联一个 Locker 接口的实例，一般我们使用 Mutex 或者 RWMutex。

```go
type Cond
  func NeWCond(l Locker) *Cond
  func (c *Cond) Broadcast()
  func (c *Cond) Signal()
  func (c *Cond) Wait()
```

**Signal 方法**：允许调用者 Caller 唤醒一个等待此 Cond 的 goroutine。如果此时没有等待的 goroutine，显然无需通知 waiter；如果 Cond 等待队列中有一个或者多个等待的 goroutine，则需要从等待队列中移除第一个 goroutine 并把它唤醒。

**Broadcast 方法**：允许调用者 Caller 唤醒所有等待此 Cond 的 goroutine。如果此时没有等待的 goroutine，显然无需通知 waiter；如果 Cond 等待队列中有一个或者多个等待的 goroutine，则清空所有等待的 goroutine，并全部唤醒。在其他编程语言中，比如 Java 语言中，Broadcast 方法也被叫做 notifyAll 方法。

**Wait 方法**：会把调用者 Caller 放入 Cond 的等待队列中并阻塞，直到被 Signal 或者 Broadcast 的方法从等待队列中移除并唤醒。

## 实现原理

```go
type Cond struct {
	noCopy  noCopy
	L       Locker      //  当观察或者修改等待条件的时候需要加锁
	notify  notifyList  // 等待队列
	checker copyChecker // 是一个辅助结构，可以在运行时检查 Cond 是否被复制使用。
}

func NewCond(l Locker) *Cond {
	return &Cond{L: l}
}

func (c *Cond) Wait() {
	c.checker.check()
	// 增加到等待队列中
	t := runtime_notifyListAdd(&c.notify)
	c.L.Unlock()
	// 阻塞休眠直到被唤醒
	runtime_notifyListWait(&c.notify, t)
	c.L.Lock()
}

func (c *Cond) Signal() {
	c.checker.check()
	runtime_notifyListNotifyOne(&c.notify)
}

func (c *Cond) Broadcast() {
	c.checker.check()
	runtime_notifyListNotifyAll(&c.notify)
}
```

**Wait 把调用者加入到等待队列时会释放锁，在被唤醒之后还会请求锁。在阻塞休眠期间，调用者是不持有锁的，这样能让其他 goroutine 有机会检查或者更新等待变量。**

## 常见错误

```go
package main

import (
	"log"
	"math/rand"
	"sync"
	"time"
)

func main() {
	cond := sync.NewCond(&sync.Mutex{})
	var ready int

	for i := 0; i < 10; i++ {
		go func(i int) {
			time.Sleep(time.Duration(rand.Int63n(10)) * time.Second)
			cond.L.Lock()
			ready++
			cond.L.Unlock()
			log.Printf("#%d", i)
			cond.Broadcast()
		}(i)
	}

	go func() {
		cond.L.Lock()
		for ready != 10 {
			cond.Wait()
			log.Println("裁判员被唤醒一次")
		}
		cond.L.Unlock()
	}()
	time.Sleep(time.Second * 10)
	log.Println("所有运动员都准备就绪。比赛开始，3，2，1, ......")
}
```

### 1.调用 Wait 的时候没有加锁

上述黑字部分即为原因，加入队列后会释放锁，所以要先获取锁。

注释代码 26 即可出现错误。

### 2.只调用了一次 Wait，没有检查等待条件是否满足，结果条件没满足，程序就继续执行了。

注释代码 27 和 30 即可出现错误。

**waiter goroutine 被唤醒不等于等待条件被满足，只是多了一次检查的机会**

## 和 Channel 对比

很多情况下，项目中更适用于 Channel，但是 Cond 也有属于自己的优势。

1. Cond 和一个 Locker 关联，可以利用这个 Locker 对相关的依赖条件更改提供保护。
2. Cond 可以同时支持 Signal 和 Broadcast 方法，而 Channel 只能同时支持其中一种。
3. Cond 的 Broadcast 方法可以被重复调用。等待条件再次变成不满足的状态后，我们又可以调用 Broadcast 再次唤醒等待的 goroutine。这也是 Channel 不能支持的，Channel 被 close 掉了之后不支持再 open。
