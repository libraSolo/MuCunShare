+++
date = '2024-8-24T16:21:57+08:00'
draft = false
title = 'Go并发_分布式(etcd)'
author = '木村凉太'
categories = 'Golang'
hiddenFromHomePage = true 
+++

# Go并发_分布式(etcd)

# 分布式

分布式（Distributed）是指将系统的组件或服务分散在多个物理或虚拟位置上，以便于处理、存储和管理数据。分布式系统通常由多个计算节点组成，这些节点通过网络进行通信和协作。

![请输入图片描述](http://mucunliangtai.com/usr/uploads/2024/08/740177789.jpg)

![请输入图片描述](http://mucunliangtai.com/usr/uploads/2024/09/1905491480.jpg)

## 选举

```go
func (e *Election) Campaign(ctx context.Context, val string) error
```

把一个节点选举为主节点，并且会设置一个值。这是一个阻塞方法，在调用它的时候会被阻塞，直到满足下面的三个条件之一，才会取消阻塞。

1. 成功当选为主
2. 此方法返回错误
3. ctx 被取消

```go
func (e *Election) Resign(ctx context.Context) (err error)
```

它的作用是，重新设置 Leader 的值，但是不会重新选主，这个方法会返回新值设置成功或者失败的信息。

```go
func (e *Election) Resign(ctx context.Context) (err error)
```

开始新一次选举。这个方法会返回新的选举成功或者失败的信息。

```go
func (e *Election) Leader(ctx context.Context) (*v3.GetResponse, error)
```

如果当前还没有 Leader，就返回一个错误，可以使用这个方法来查询主节点信息。

```go
func (e *Election) Rev() int64
```

每次主节点的变动都会生成一个新的版本号，可以查询版本号信息。

```go
func (e *Election) Observe(ctx context.Context) <-chan v3.GetResponse
```

返回主节点最近一次变动的信息。

## 互斥锁

### Locker

ETCD 提供了简单的 Locker，提供了 Lock/UnLock

```go
func NewLocker(s *Session, pfx string) sync.Locker
```

使用demo

```go
package main

import (
    "flag"
    "log"
    "math/rand"
    "strings"
    "time"

    "github.com/coreos/etcd/clientv3"
    "github.com/coreos/etcd/clientv3/concurrency"
)

var (
    addr     = flag.String("addr", "http://127.0.0.1:2379", "etcd addresses")
    lockName = flag.String("name", "my-test-lock", "lock name")
)

func main() {
    flag.Parse()
  
    rand.Seed(time.Now().UnixNano())
    // etcd地址
    endpoints := strings.Split(*addr, ",")
    // 生成一个etcd client
    cli, err := clientv3.New(clientv3.Config{Endpoints: endpoints})
    if err != nil {
        log.Fatal(err)
    }
    defer cli.Close()
    useLock(cli) // 测试锁
}

func useLock(cli *clientv3.Client) {
    // 为锁生成session
    s1, err := concurrency.NewSession(cli)
    if err != nil {
        log.Fatal(err)
    }
    defer s1.Close()
    //得到一个分布式锁
    locker := concurrency.NewLocker(s1, *lockName)

    // 请求锁
    log.Println("acquiring lock")
    locker.Lock()
    log.Println("acquired lock")

    // 等待一段时间
    time.Sleep(time.Duration(rand.Intn(30)) * time.Second)
    locker.Unlock() // 释放锁

    log.Println("released lock")
}
```

### Mutex

```go
func useMutex(cli *clientv3.Client) {
    // 为锁生成session
    s1, err := concurrency.NewSession(cli)
    if err != nil {
        log.Fatal(err)
    }
    defer s1.Close()
    m1 := concurrency.NewMutex(s1, *lockName)

    //在请求锁之前查询key
    log.Printf("before acquiring. key: %s", m1.Key())
    // 请求锁
    log.Println("acquiring lock")
    if err := m1.Lock(context.TODO()); err != nil {
        log.Fatal(err)
    }
    log.Printf("acquired lock. key: %s", m1.Key())

    //等待一段时间
    time.Sleep(time.Duration(rand.Intn(30)) * time.Second)

    // 释放锁
    if err := m1.Unlock(context.TODO()); err != nil {
        log.Fatal(err)
    }
    log.Println("released lock")
}
```

### 读写锁

```go
package main


import (
    "bufio"
    "flag"
    "fmt"
    "log"
    "math/rand"
    "os"
    "strings"
    "time"

    "github.com/coreos/etcd/clientv3"
    "github.com/coreos/etcd/clientv3/concurrency"
    recipe "github.com/coreos/etcd/contrib/recipes"
)

var (
    addr     = flag.String("addr", "http://127.0.0.1:2379", "etcd addresses")
    lockName = flag.String("name", "my-test-lock", "lock name")
    action   = flag.String("rw", "w", "r means acquiring read lock, w means acquiring write lock")
)


func main() {
    flag.Parse()
    rand.Seed(time.Now().UnixNano())

    // 解析etcd地址
    endpoints := strings.Split(*addr, ",")

    // 创建etcd的client
    cli, err := clientv3.New(clientv3.Config{Endpoints: endpoints})
    if err != nil {
        log.Fatal(err)
    }
    defer cli.Close()
    // 创建session
    s1, err := concurrency.NewSession(cli)
    if err != nil {
        log.Fatal(err)
    }
    defer s1.Close()
    m1 := recipe.NewRWMutex(s1, *lockName)

    // 从命令行读取命令
    consolescanner := bufio.NewScanner(os.Stdin)
    for consolescanner.Scan() {
        action := consolescanner.Text()
        switch action {
        case "w": // 请求写锁
            testWriteLocker(m1)
        case "r": // 请求读锁
            testReadLocker(m1)
        default:
            fmt.Println("unknown action")
        }
    }
}

func testWriteLocker(m1 *recipe.RWMutex) {
    // 请求写锁
    log.Println("acquiring write lock")
    if err := m1.Lock(); err != nil {
        log.Fatal(err)
    }
    log.Println("acquired write lock")

    // 等待一段时间
    time.Sleep(time.Duration(rand.Intn(10)) * time.Second)

    // 释放写锁
    if err := m1.Unlock(); err != nil {
        log.Fatal(err)
    }
    log.Println("released write lock")
}

func testReadLocker(m1 *recipe.RWMutex) {
    // 请求读锁
    log.Println("acquiring read lock")
    if err := m1.RLock(); err != nil {
        log.Fatal(err)
    }
    log.Println("acquired read lock")

    // 等待一段时间
    time.Sleep(time.Duration(rand.Intn(10)) * time.Second)

    // 释放写锁
    if err := m1.RUnlock(); err != nil {
        log.Fatal(err)
    }
    log.Println("released read lock")
}
```

## etcd 队列

* 分布式队列
* 优先级队列

### 分布式队列

```go
func NewQueue(client *v3.Client, keyPrefix string) *Queue

// 入队
func (q *Queue) Enqueue(val string) error
//出队
func (q *Queue) Dequeue() (string, error)
```

分布式队列当前为空，调用 Dequeue 方法的话，会被阻塞，直到有元素可以出队才返回。

demo:

```go

package main


import (
    "bufio"
    "flag"
    "fmt"
    "log"
    "os"
    "strings"


    "github.com/coreos/etcd/clientv3"
    recipe "github.com/coreos/etcd/contrib/recipes"
)


var (
    addr      = flag.String("addr", "http://127.0.0.1:2379", "etcd addresses")
    queueName = flag.String("name", "my-test-queue", "queue name")
)


func main() {
    flag.Parse()


    // 解析etcd地址
    endpoints := strings.Split(*addr, ",")


    // 创建etcd的client
    cli, err := clientv3.New(clientv3.Config{Endpoints: endpoints})
    if err != nil {
        log.Fatal(err)
    }
    defer cli.Close()


    // 创建/获取队列
    q := recipe.NewQueue(cli, *queueName)


    // 从命令行读取命令
    consolescanner := bufio.NewScanner(os.Stdin)
    for consolescanner.Scan() {
        action := consolescanner.Text()
        items := strings.Split(action, " ")
        switch items[0] {
        case "push": // 加入队列
            if len(items) != 2 {
                fmt.Println("must set value to push")
                continue
            }
            q.Enqueue(items[1]) // 入队
        case "pop": // 从队列弹出
            v, err := q.Dequeue() // 出队
            if err != nil {
                log.Fatal(err)
            }
            fmt.Println(v) // 输出出队的元素
        case "quit", "exit": //退出
            return
        default:
            fmt.Println("unknown action")
        }
    }
}
```

### 优先级队列

多提供一个 uint16 的整数，作为优先级。

## 分布式栅栏

* Barrier：分布式栅栏。如果持有 Barrier 的节点释放了它，所有等待这个 Barrier 的节点就不会被阻塞，而是会继续执行。
* DoubleBarrier：计数型栅栏。在初始化计数型栅栏的时候，我们就必须提供参与节点的数量，当这些数量的节点都 Enter 或者 Leave 的时候，这个栅栏就会放开。所以，我们把它称为计数型栅栏。

### 分布式栅栏

```go
func NewBarrier(client *v3.Client, key string) *Barrier

func (b *Barrier) Hold() error // 创建一个 Barrier, 如果已经创建好，有节点调用它的 Wait 就会被阻塞。
func (b *Barrier) Release() error // 释放 Barrier，所有阻塞的节点都被放行。
func (b *Barrier) Wait() error // 阻塞当前调用者，直到 Barrier 被释放。如果栅栏不存在，则不会阻塞。
```

### 计数型栅栏

```go
func NewDoubleBarrier(s *concurrency.Session, key string, count int) *DoubleBarrier

func (b *DoubleBarrier) Enter() error // 当调用者 Enter 则被阻塞。直到一共 count 个节点调用了 Enter，这个 count 个被阻塞的节点才会继续执行。
func (b *DoubleBarrier) Leave() error // 节点调用 Leave 方法的时候，会被阻塞，直到有 count 个节点，都调用了 Leave 方法，这些节点才能继续执行。
```

## STM

事务：要么全部成功，要么全失败。
etcd 的事务实现方式是基于 CAS 方式实现的，融合了 Get、Put 和 Delete 操作。

etcd 的事务操作如下，分为条件块、成功块和失败块，条件块用来检测事务是否成功，如果成功，就执行 Then(...)，如果失败，就执行 Else(...)：

```go
Txn().If(cond1, cond2, ...).Then(op1, op2, ...,).Else(op1’, op2’, …)
```

转账例子：

```go
func doTxnXfer(etcd *v3.Client, from, to string, amount uint) (bool, error) {
    // 一个查询事务
    getresp, err := etcd.Txn(ctx.TODO()).Then(OpGet(from), OpGet(to)).Commit()
    if err != nil {
         return false, err
    }
    // 获取转账账户的值
    fromKV := getresp.Responses[0].GetRangeResponse().Kvs[0]
    toKV := getresp.Responses[1].GetRangeResponse().Kvs[1]
    fromV, toV := toUInt64(fromKV.Value), toUint64(toKV.Value)
    if fromV < amount {
        return false, fmt.Errorf(“insufficient value”)
    }
    // 转账事务
    // 条件块
    txn := etcd.Txn(ctx.TODO()).If(
        v3.Compare(v3.ModRevision(from), “=”, fromKV.ModRevision),
        v3.Compare(v3.ModRevision(to), “=”, toKV.ModRevision))
    // 成功块
    txn = txn.Then(
        OpPut(from, fromUint64(fromV - amount)),
        OpPut(to, fromUint64(toV + amount))
    //提交事务 
    putresp, err := txn.Commit()
    // 检查事务的执行结果
    if err != nil {
        return false, err
    }
    return putresp.Succeeded, nil
}
```

etcd 封装了更简单的 STM，首先分装一个 apply 函数，这个函数的执行是在一个事务之中，包含了一个 STM 类型的参数

```go
apply func(STM) error
```

STM 提供了 4 个方法

```go
type STM interface {
  Get(key ...string) string
  Put(key, val string, opts ...v3.OpOption)
  Rev(key string) int64
  Del(key string)
}
```

下面这个例子创建了 5 个银行账号，然后随机选择一些账号两两转账。在转账的时候，要把源账号一半的钱要转给目标账号。这个例子的代码如下：

```go
package main


import (
    "context"
    "flag"
    "fmt"
    "log"
    "math/rand"
    "strings"
    "sync"


    "github.com/coreos/etcd/clientv3"
    "github.com/coreos/etcd/clientv3/concurrency"
)


var (
    addr = flag.String("addr", "http://127.0.0.1:2379", "etcd addresses")
)


func main() {
    flag.Parse()


    // 解析etcd地址
    endpoints := strings.Split(*addr, ",")


    cli, err := clientv3.New(clientv3.Config{Endpoints: endpoints})
    if err != nil {
        log.Fatal(err)
    }
    defer cli.Close()


    // 设置5个账户，每个账号都有100元，总共500元
    totalAccounts := 5
    for i := 0; i < totalAccounts; i++ {
        k := fmt.Sprintf("accts/%d", i)
        if _, err = cli.Put(context.TODO(), k, "100"); err != nil {
            log.Fatal(err)
        }
    }


    // STM的应用函数，主要的事务逻辑
    exchange := func(stm concurrency.STM) error {
        // 随机得到两个转账账号
        from, to := rand.Intn(totalAccounts), rand.Intn(totalAccounts)
        if from == to {
            // 自己不和自己转账
            return nil
        }
        // 读取账号的值
        fromK, toK := fmt.Sprintf("accts/%d", from), fmt.Sprintf("accts/%d", to)
        fromV, toV := stm.Get(fromK), stm.Get(toK)
        fromInt, toInt := 0, 0
        fmt.Sscanf(fromV, "%d", &fromInt)
        fmt.Sscanf(toV, "%d", &toInt)


        // 把源账号一半的钱转账给目标账号
        xfer := fromInt / 2
        fromInt, toInt = fromInt-xfer, toInt+xfer


        // 把转账后的值写回
        stm.Put(fromK, fmt.Sprintf("%d", fromInt))
        stm.Put(toK, fmt.Sprintf("%d", toInt))
        return nil
    }


    // 启动10个goroutine进行转账操作
    var wg sync.WaitGroup
    wg.Add(10)
    for i := 0; i < 10; i++ {
        go func() {
            defer wg.Done()
            for j := 0; j < 100; j++ {
                if _, serr := concurrency.NewSTM(cli, exchange); serr != nil {
                    log.Fatal(serr)
                }
            }
        }()
    }
    wg.Wait()


    // 检查账号最后的数目
    sum := 0
    accts, err := cli.Get(context.TODO(), "accts/", clientv3.WithPrefix()) // 得到所有账号
    if err != nil {
        log.Fatal(err)
    }
    for _, kv := range accts.Kvs { // 遍历账号的值
        v := 0
        fmt.Sscanf(string(kv.Value), "%d", &v)
        sum += v
        log.Printf("account %s: %d", kv.Key, v)
    }


    log.Println("account sum is", sum) // 总数
}