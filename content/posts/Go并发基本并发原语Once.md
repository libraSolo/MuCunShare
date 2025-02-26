# Go并发_基本并发原语_Once

# Once

![请输入图片描述](http://mucunliangtai.com/usr/uploads/2024/08/2051938637.jpg)

Once 常常用来初始化单例资源，或者并发访问只需初始化一次的共享资源，或者在测试的时候初始化一次测试资源。

## 方法

```go
func (o *Once) Do(f func())
```

可以多次调用 Do 方法，但是只有第一次调用 Do 方法时 f 参数才会执行，这里的 f 是一个无参数无返回值的函数。

## 常见错误 （罕见）

### 1. 死锁

Do 方法会执行一次 f，但是如果 f 中再次调用这个 Once 的 Do 方法的话，就会导致死锁的情况出现。

```go
func main() {
    var once sync.Once
    once.Do(func() {
        once.Do(func() {
            fmt.Println("初始化")
        })
    })
}
```

### 2.未初始化

如果执行 f 的时候 panic，未初始化成功，但是 Once 依旧认为执行成功，不会再次执行 f。

优化 Once 的并发原语，增加了 f 的 error 返回和 Once 的 error 返回，通过判断是否为 nil 来看是否将 done 设为 1。

```go
// 一个功能更加强大的Once
type Once struct {
    m    sync.Mutex
    done uint32
}
// 传入的函数f有返回值error，如果初始化失败，需要返回失败的error
// Do方法会把这个error返回给调用者
func (o *Once) Do(f func() error) error {
    if atomic.LoadUint32(&o.done) == 1 { //fast path
        return nil
    }
    return o.slowDo(f)
}
// 如果还没有初始化
func (o *Once) slowDo(f func() error) error {
    o.m.Lock()
    defer o.m.Unlock()
    var err error
    if o.done == 0 { // 双检查，还没有初始化
        err = f()
        if err == nil { // 初始化成功才将标记置为已初始化
            atomic.StoreUint32(&o.done, 1)
        }
    }
    return err
}