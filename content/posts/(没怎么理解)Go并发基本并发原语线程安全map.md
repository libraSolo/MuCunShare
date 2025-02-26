# (没怎么理解)Go并发_基本并发原语_线程安全map

# sync.Map

![请输入图片描述](http://mucunliangtai.com/usr/uploads/2024/08/502003932.jpg)

`map[k]v` 其中，k是可比较类型，v是任意。

**可比较类型**： 可以通过 == 或者 != 比较的。

sync.Map 并不是用来替换内建的 map 类型的，它只能被应用在一些特殊的场景里。

## 场景应用

### 1. 只会增长的缓存系统中，一个 key 只写入一次而被读很多次

### 2. 多个 goroutine 为不相交的键集读、写和重写键值对。

## 源码解析

```go
type Map struct {
    mu Mutex
    // 基本上你可以把它看成一个安全的只读的map
    // 它包含的元素其实也是通过原子操作更新的，但是已删除的entry就需要加锁操作了
    read atomic.Value // readOnly

    // 包含需要加锁才能访问的元素
    // 包括所有在read字段中但未被expunged（删除）的元素以及新加的元素
    dirty map[interface{}]*entry

    // 记录从read中读取miss的次数，一旦miss数和dirty长度一样了，就会把dirty提升为read，并把dirty置空
    misses int
}

type readOnly struct {
    m       map[interface{}]*entry
    amended bool // 当dirty中包含read没有的数据时为true，比如新增一条数据
}

// expunged是用来标识此项已经删掉的指针
// 当map中的一个项目被删除了，只是把它的值标记为expunged，以后才有机会真正删除此项
var expunged = unsafe.Pointer(new(interface{}))

// entry代表一个值
type entry struct {
    p unsafe.Pointer // *interface{}
}