+++
date = '2024-8-24T16:21:57+08:00'
draft = false
title = 'Go并发_Semaphore'
author = '木村凉太'
categories = 'Golang'
hiddenFromHomePage = true 
+++


# Go并发_Semaphore

# Semaphore

![请输入图片描述](http://mucunliangtai.com/usr/uploads/2024/09/3849264388.jpg)

* 信号量
  1. 计数信号量
  2. 二进位信号量(计数信号量仅为0或者1)
     有时候，互斥锁也会用二进位信号量

## Go 中信号量

Weighted 包

```go
func NewWeighted(n int64) *Weighted
func (s *Weighted) Acquire(ctx context.Context, n int64) error // 相当于 P 操作，你可以一次获取多个资源，如果没有足够多的资源，调用者就会被阻塞。
func (s *Weighted) Release(n int64) // 相当于 V 操作， 释放 n 个资源
func (s *Weighted) TryAcquire(n int64) bool // 尝试获取 n 个资源，不会阻塞，失败返回 false。
```

测试用例：

```go
var (
    maxWorkers = runtime.GOMAXPROCS(0)                    // worker数量
    sema       = semaphore.NewWeighted(int64(maxWorkers)) //信号量
    task       = make([]int, maxWorkers*4)                // 任务数，是worker的四倍
)

func main() {
    ctx := context.Background()

    for i := range task {
        // 如果没有worker可用，会阻塞在这里，直到某个worker被释放
        if err := sema.Acquire(ctx, 1); err != nil {
            break
        }

        // 启动worker goroutine
        go func(i int) {
            defer sema.Release(1)
            time.Sleep(100 * time.Millisecond) // 模拟一个耗时操作
            task[i] = i + 1
        }(i)
    }

    // 请求所有的worker,这样能确保前面的worker都执行完
    if err := sema.Acquire(ctx, int64(maxWorkers)); err != nil {
        log.Printf("获取所有的worker失败: %v", err)
    }

    fmt.Println(task)
}
```

### Weighted

使用互斥锁 + 等待队列，通过 channel 来实现的

```go
type Weighted struct {
    size    int64         // 最大资源数
    cur     int64         // 当前已被使用的资源
    mu      sync.Mutex    // 互斥锁，对字段的保护
    waiters list.List     // 等待队列
}