+++
date = '2024-8-10T16:21:57+08:00'
draft = false
title = 'Go并发_基本并发原语_Context'
author = '木村凉太'
categories = 'Golang'
hiddenFromHomePage = true 
+++


# Go并发_基本并发原语_Context

# Context

上下文联系。

![请输入图片描述](http://mucunliangtai.com/usr/uploads/2024/08/2611943546.jpg)

## 使用场景

1. 上下文信息传递 （request-scoped），比如处理 http 请求、在请求处理链路上传递信息
2. 控制子 goroutine 的运行
3. 超时控制调用
4. 取消的方法调用

## 基本使用

```go
type Context interface {
	Deadline() (deadline time.Time, ok bool) // 方法会返回这个 Context 被取消的截止日期。如果没有设置截止日期，ok 的值是 false。
	Done() <-chan struct{}                   // 方法返回一个 Channel 对象。在 Context 被取消时，此 Channel 会被 close，如果没被取消，可能会返回 nil。
	Err() error                              //
	Value(key interface{}) interface{}       // 返回此 ctx 中和指定的 key 相关联的 value。
}

```

### 生成顶层 Context 的方法、

* context.Background()：返回一个非 nil 的、空的 Context，没有任何值，不会被 cancel，不会超时，没有截止日期。一般用在主函数、初始化、测试以及创建根 Context 的时候。
* context.TODO()：Background() 的别名，不知道用啥或者不知道什么上下文用这个。

### 默认原则

1. 一般函数使用 Context 的时候，会把这个参数放在第一个参数的位置。
2. 从来不把 nil 当做 Context 类型的参数值，可以用 context.TODO()。
3. Context 只用来临时做函数之间的上下文透传，不能持久化 Context 或者把 Context 长久保存。
4. **key 的类型不应该是字符串类型或者其它内建类型，否则容易在包之间使用 Context 时候产生冲突。使用 WithValue 时，key 的类型应该是自己定义的类型。**
5. 常常使用 struct{}作为底层类型定义 key 的类型。对于 exported key 的静态类型，常常是接口或者指针。这样可以尽量减少内存分配。

### WithValue

它持有一个 key-value 键值对，还持有 parent 的 Context。它覆盖了 Value 方法，优先从自己的存储中检查这个 key，不存在的话会从 parent 中继续检查，链式查找。

```go
ctx = context.TODO()
ctx = context.WithValue(ctx, "key1", "0001")
ctx = context.WithValue(ctx, "key2", "0001")
ctx = context.WithValue(ctx, "key3", "0001")
ctx = context.WithValue(ctx, "key4", "0004")

fmt.Println(ctx.Value("key1"))
```

查找key -> key4 -> key3 -> key2 -> key1

### WithCancel

```go
	ctx, cancelFunc := context.WithCancel()
	defer cancelFunc()
```

**任务完成和中途放弃都需要调用 cancelFunc(), context 才会释放资源。**

cancel 是向下传递，其依赖的子孙都会被 cancel，但是不会向上传递。

被取消时，err 字段为 `var Canceled = errors.New("context canceled")` 错误。

### WithDeadline

WithDeadline 会返回一个 parent 的副本，并且设置了一个不晚于参数 d 的截止时间。

timerCtx 的 Done 被 Close 掉的原因。

1. 截止时间到了
2. cancel 函数被调用
3. parent 的 Done 被 close

**同 cancelCtx 一样，cancelFunc() 一定要调用。**

### WithTimeout

同 WithDeadline ，一个是截至时间，一个是超时时间。