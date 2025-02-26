# Go并发_分组操作



# Go并发_分组操作

# 分组操作

分组执行一批相同或类似的任务。

# ErrGroup

大任务拆成几个小任务并发执行

## WithContext

```go
func WithContext(ctx context.Context) (*Group, context.Context)
```

这个方法返回一个 Group 实例，同时还会返回一个使用 context.WithCancel(ctx) 生成的新 Context。一旦有一个子任务返回错误，或者是 Wait 调用返回，这个新 Context 就会被 cancel。

如果传递给 WithContext 的 ctx 参数，是一个可以 cancel 的 Context 的话，那么，它被 cancel 的时候，并不会终止正在执行的子任务。(可以由子任务终止，向上传递)

## Go

```go
func (g *Group) Go(f func() error)
```

任务成功返回 nil，否则返回 error，并且会 cancel 那个新的 Context。

## Wait

```go
func (g *Group) Wait() error
```

类似于 WaitGroup，等所有的子任务完成后，就会返回，否则阻塞等待。如果有多个子任务返回错误，它只会返回第一个出现的错误。

## Pipeline 任务流水线

```go
package main

import (
   ......
    "golang.org/x/sync/errgroup"
)

// 一个多阶段的pipeline.使用有限的goroutine计算每个文件的md5值.
func main() {
    m, err := MD5All(context.Background(), ".")
    if err != nil {
        log.Fatal(err)
    }

    for k, sum := range m {
        fmt.Printf("%s:\t%x\n", k, sum)
    }
}

type result struct {
    path string
    sum  [md5.Size]byte
}

// 遍历根目录下所有的文件和子文件夹,计算它们的md5的值.
func MD5All(ctx context.Context, root string) (map[string][md5.Size]byte, error) {
    g, ctx := errgroup.WithContext(ctx)
    paths := make(chan string) // 文件路径channel

    g.Go(func() error {
        defer close(paths) // 遍历完关闭paths chan
        return filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
            ...... //将文件路径放入到paths
            return nil
        })
    })

    // 启动20个goroutine执行计算md5的任务，计算的文件由上一阶段的文件遍历子任务生成.
    c := make(chan result)
    const numDigesters = 20
    for i := 0; i < numDigesters; i++ {
        g.Go(func() error {
            for path := range paths { // 遍历直到paths chan被关闭
                ...... // 计算path的md5值，放入到c中
            }
            return nil
        })
    }
    go func() {
        g.Wait() // 20个goroutine以及遍历文件的goroutine都执行完
        close(c) // 关闭收集结果的chan
    }()


    m := make(map[string][md5.Size]byte)
    for r := range c { // 将md5结果从chan中读取到map中,直到c被关闭才退出
        m[r.path] = r.sum
    }

    // 再次调用Wait，依然可以得到group的error信息
    if err := g.Wait(); err != nil {
        return nil, err
    }
    return m, nil
}
