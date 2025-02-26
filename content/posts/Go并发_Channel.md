+++
date = '2024-8-24T16:21:57+08:00'
draft = false
title = 'Go并发_Channel'
author = '木村凉太'
categories = 'Golang'
hiddenFromHomePage = true 
+++

# Go并发_Channel

# Channel

![请输入图片描述](http://mucunliangtai.com/usr/uploads/2024/08/506958358.jpg)

channel 是一种用于在 Goroutine 之间进行通信和同步的强大工具。它允许一个 Goroutine 将数据发送到另一个 Goroutine，从而实现并发编程中的协作。

## 使用场景

1. 数据交流
   当作并发的 buffer 或者 queue，解决生产者 - 消费者问题。
2. 数据传递
   一个 goroutine 将数据交给另一个 goroutine，相当于把数据的拥有权 (引用) 托付出去。
3. 信号通知
   一个 goroutine 可以将信号 (closing、closed、data ready 等) 传递给另一个或者另一组 goroutine。
4. 任务编排
   可以让一组 goroutine 按照一定的顺序并发或者串行的执行，这就是编排的功能。
5. 锁
   实现互斥锁的机制。

## 基本用法

其类型可以分为三种：
只能接收，只能发送，既可以接也可以发

```go
chan string          // 可以发送接收string
chan<- struct{}      // 只能发送struct{}
<-chan int           // 只能从chan接收int
```

### 初始化

`make(chan int, 100)` or `make(chan int)`

未初始化的 chan 的零值是 nil，是一种特殊的 chan，对值是 nil 的 chan 的发送/接收/调用者总是会阻塞。

## 实现原理

```go
type hchan struct {
	qcount   uint           // 代表 chan 中已经接收但还没被取走的元素的个数。内建函数 len 可以返回这个字段的值。
	dataqsiz uint           // 循环队列的大小
	buf      unsafe.Pointer // 循环队列的指针
	elemsize uint16         // chan 中元素大小
	closed   uint32         // 是否已经被 close
	elemtype *_type         // chan 中元素类型
	sendx    uint           // 处理发送数据的指针在 buf 中的位置。buf 的总大小是 elemsize 的整数倍，而且 buf 是一个循环列表。
	recvx    uint           // 处理接收请求时的指针在 buf 中的位置。
	recvq    waitq          // chan 是多生产者多消费者的模式，如果消费者因为没有数据可读而被阻塞了，就会被加入到 recvq 队列中。
	sendq    waitq          // 如果生产者因为 buf 满了而阻塞，会被加入到 sendq 队列中。

	lock mutex 				// 互斥锁
}
```

### make

```go
func makechan(t *chantype, size int) *hchan {
    elem := t.elem
  
        // 略去检查代码
        mem, overflow := math.MulUintptr(elem.size, uintptr(size))
  
    //
    var c *hchan
    switch {
    case mem == 0:
      // chan的size或者元素的size是0，不必创建buf
      c = (*hchan)(mallocgc(hchanSize, nil, true))
      c.buf = c.raceaddr()
    case elem.ptrdata == 0:
      // 元素不是指针，分配一块连续的内存给hchan数据结构和buf
      c = (*hchan)(mallocgc(hchanSize+mem, nil, true))
            // hchan数据结构后面紧接着就是buf
      c.buf = add(unsafe.Pointer(c), hchanSize)
    default:
      // 元素包含指针，那么单独分配buf
      c = new(hchan)
      c.buf = mallocgc(mem, elem, true)
    }
  
        // 元素大小、类型、容量都记录下来
    c.elemsize = uint16(elem.size)
    c.elemtype = elem
    c.dataqsiz = uint(size)
    lockInit(&c.lock, lockRankHchan)

    return c
  }
```

关注是否容量为 0 ， 在判断其元素是否为 指针，同时记录元素。

### send

1. chan 为 nil，永久阻塞
2. chan 为满，不阻塞返回 false
3. chan 被 close，直接panic
4. 如果等待队列有 receiver，直接从队列弹出并交给它，不需要放入buf
5. 没有 receiver 等待，放入 buf 中并成功返回
6. buf 为满，发送者阻塞等待被唤醒或者 chan 被close。

### recv

1. chan 为 nil，永久阻塞
2. 略
3. chan 被 close，返回 true，false
4. buf 满，如果是 unbuffer 的 chan，直接将 sender 的数据交付给 receiver，否则从队头读取一个值，并把 sender 的值放在队尾。
5. 没有等待的 sender，如果 buf 有元素，拿第一个给 receiver。
6. buf 中没有元素，当前 receiver 被阻塞，直到 sender 发送数据，或者 chan 被关闭。

### close

关闭 chan。

1. 如果 chan 为 nil，close 会 panic
2. chan 已经被关闭， 再次 close 会 panic
3. chan 不为 nil，也没被关闭，就会把 sender 和 receiver 从队列中全部移除并唤醒。

## 易错场景

### 1. panic 的情况

1. close 为 nil 的 chan
2. 向为 close 的 chan 发送数据
3. close 已经 close 的 chan

### 2. 内存泄漏

对于 unbuffer 的 chan， 子协程写入，主协程读取，如果主协程未读取提前退出，子协程将阻塞。

## channel 和 锁的选择

1. 共享资源的并发访问使用传统并发原语
2. 复杂的任务编排和消息传递使用 Channel
3. 消息通知机制使用 Channel，除非只想 signal 一个 goroutine，才使用 Cond
4. 简单等待所有任务的完成用 WaitGroup，也有 Channel 的推崇者用 Channel，都可以
5. 需要和 Select 语句结合，使用 Channel
6. 需要和超时配合时，使用 Channel 和 Context。

## 利用反射处理不定数量的 chan

```go
func main() {
    var ch1 = make(chan int, 10)
    var ch2 = make(chan int, 10)

    // 创建SelectCase
    var cases = createCases(ch1, ch2)

    // 执行10次select
    for i := 0; i < 10; i++ {
        chosen, recv, ok := reflect.Select(cases)
        if recv.IsValid() { // recv case
            fmt.Println("recv:", cases[chosen].Dir, recv, ok)
        } else { // send case
            fmt.Println("send:", cases[chosen].Dir, ok)
        }
    }
}

func createCases(chs ...chan int) []reflect.SelectCase {
    var cases []reflect.SelectCase


    // 创建recv case
    for _, ch := range chs {
        cases = append(cases, reflect.SelectCase{
            Dir:  reflect.SelectRecv,
            Chan: reflect.ValueOf(ch),
        })
    }

    // 创建send case
    for i, ch := range chs {
        v := reflect.ValueOf(i)
        cases = append(cases, reflect.SelectCase{
            Dir:  reflect.SelectSend,
            Chan: reflect.ValueOf(ch),
            Send: v,
        })
    }

    return cases
}
```

## 任务编排

### Or-Done 模式

如果有多个任务，只要有任意一个任务执行完，我们就想获得这个信号，这就是 Or-Done 模式。

```go
func or(channels ...<-chan interface{}) <-chan interface{} {
    // 特殊情况，只有零个或者1个chan
    switch len(channels) {
    case 0:
        return nil
    case 1:
        return channels[0]
    }

    orDone := make(chan interface{})
    go func() {
        defer close(orDone)

        switch len(channels) {
        case 2: // 2个也是一种特殊情况
            select {
            case <-channels[0]:
            case <-channels[1]:
            }
        default: //超过两个，二分法递归处理
            m := len(channels) / 2
            select {
            case <-or(channels[:m]...):
            case <-or(channels[m:]...):
            }
        }
    }()

    return orDone
}
```

```go
func or(channels ...<-chan interface{}) <-chan interface{} {
    //特殊情况，只有0个或者1个
    switch len(channels) {
    case 0:
        return nil
    case 1:
        return channels[0]
    }

    orDone := make(chan interface{})
    go func() {
        defer close(orDone)
        // 利用反射构建SelectCase
        var cases []reflect.SelectCase
        for _, c := range channels {
            cases = append(cases, reflect.SelectCase{
                Dir:  reflect.SelectRecv,
                Chan: reflect.ValueOf(c),
            })
        }

        // 随机选择一个可用的case
        reflect.Select(cases)
    }()


    return orDone
}
```

### 扇入模式

多个输入，一个输出。

```go
func fanInReflect(chans ...<-chan interface{}) <-chan interface{} {
    out := make(chan interface{})
    go func() {
        defer close(out)
        // 构造SelectCase slice
        var cases []reflect.SelectCase
        for _, c := range chans {
            cases = append(cases, reflect.SelectCase{
                Dir:  reflect.SelectRecv,
                Chan: reflect.ValueOf(c),
            })
        }
  
        // 循环，从cases中选择一个可用的
        for len(cases) > 0 {
            i, v, ok := reflect.Select(cases)
            if !ok { // 此channel已经close
                cases = append(cases[:i], cases[i+1:]...)
                continue
            }
            out <- v.Interface()
        }
    }()
    return out
}
```

```go
func fanInRec(chans ...<-chan interface{}) <-chan interface{} {
    switch len(chans) {
    case 0:
        c := make(chan interface{})
        close(c)
        return c
    case 1:
        return chans[0]
    case 2:
        return mergeTwo(chans[0], chans[1])
    default:
        m := len(chans) / 2
        return mergeTwo(
            fanInRec(chans[:m]...),
            fanInRec(chans[m:]...))
    }
}

// mergeTwo 的方法，是将两个 Channel 合并成一个 Channel，是扇入形式的一种特例（只处理两个 Channel）。 下面我来借助一段代码帮你理解下这个方法。
func mergeTwo(a, b <-chan interface{}) <-chan interface{} {
    c := make(chan interface{})
    go func() {
        defer close(c)
        for a != nil || b != nil { //只要还有可读的chan
            select {
            case v, ok := <-a:
                if !ok { // a 已关闭，设置为nil
                    a = nil
                    continue
                }
                c <- v
            case v, ok := <-b:
                if !ok { // b 已关闭，设置为nil
                    b = nil
                    continue
                }
                c <- v
            }
        }
    }()
    return c
}
```

### 扇出模式

一个输入源 Channel，有多个目标 Channel

```go
func fanOut(ch <-chan interface{}, out []chan interface{}, async bool) {
    go func() {
        defer func() { //退出时关闭所有的输出chan
            for i := 0; i < len(out); i++ {
                close(out[i])
            }
        }()

        for v := range ch { // 从输入chan中读取数据
            v := v
            for i := 0; i < len(out); i++ {
                i := i
                if async { //异步
                    go func() {
                        out[i] <- v // 放入到输出chan中,异步方式
                    }()
                } else {
                    out[i] <- v // 放入到输出chan中，同步方式
                }
            }
        }
    }()
}
```

### Stream

把 Channel 当作流式管道使用的方式，也就是把 Channel 看作流。

把 slice 转换成流

```go
func asStream(done <-chan struct{}, values ...interface{}) <-chan interface{} {
    s := make(chan interface{}) //创建一个unbuffered的channel
    go func() { // 启动一个goroutine，往s中塞数据
        defer close(s) // 退出时关闭chan
        for _, v := range values { // 遍历数组
            select {
            case <-done:
                return
            case s <- v: // 将数组元素塞入到chan中
            }
        }
    }()
    return s
}
```

1. takeN：只取流中的前 n 个数据；
2. takeFn：筛选流中的数据，只保留满足条件的数据；
3. takeWhile：只取前面满足条件的数据，一旦不满足条件，就不再取；
4. skipN：跳过流中前几个数据；
5. skipFn：跳过满足条件的数据；
6. skipWhile：跳过前面满足条件的数据，一旦不满足条件，当前这个元素和以后的元素都会输出给 Channel 的 receiver。

```go
func takeN(done <-chan struct{}, valueStream <-chan interface{}, num int) <-chan interface{} {
    takeStream := make(chan interface{}) // 创建输出流
    go func() {
        defer close(takeStream)
        for i := 0; i < num; i++ { // 只读取前num个元素
            select {
            case <-done:
                return
            case takeStream <- <-valueStream: //从输入流中读取元素
            }
        }
    }()
    return takeStream
}
```

### map-reduce(单机单进程版)

map-reduce 分为两个步骤，第一步是映射（map），处理队列中的数据，第二步是规约（reduce），把列表中的每一个元素按照一定的处理方式处理成结果，放入到结果队列中。

```go
func mapChan(in <-chan interface{}, fn func(interface{}) interface{}) <-chan interface{} {
    out := make(chan interface{}) //创建一个输出chan
    if in == nil { // 异常检查
        close(out)
        return out
    }

    go func() { // 启动一个goroutine,实现map的主要逻辑
        defer close(out)
        for v := range in { // 从输入chan读取数据，执行业务操作，也就是map操作
            out <- fn(v)
        }
    }()

    return out
}
```

```go
func reduce(in <-chan interface{}, fn func(r, v interface{}) interface{}) interface{} {
    if in == nil { // 异常检查
        return nil
    }

    out := <-in // 先读取第一个元素
    for v := range in { // 实现reduce的主要逻辑
        out = fn(out, v)
    }

    return out
}
```

例子

```go
// 生成一个数据流
func asStream(done <-chan struct{}) <-chan interface{} {
    s := make(chan interface{})
    values := []int{1, 2, 3, 4, 5}
    go func() {
        defer close(s)
        for _, v := range values { // 从数组生成
            select {
            case <-done:
                return
            case s <- v:
            }
        }
    }()
    return s
}

func main() {
    in := asStream(nil)

    // map操作: 乘以10
    mapFn := func(v interface{}) interface{} {
        return v.(int) * 10
    }

    // reduce操作: 对map的结果进行累加
    reduceFn := func(r, v interface{}) interface{} {
        return r.(int) + v.(int)
    }

    sum := reduce(mapChan(in, mapFn), reduceFn) //返回累加结果
    fmt.Println(sum)
}