# Go并发心法 - 分身之术

## 一、Goroutine - 分身术

Goroutine是Go的轻量级线程，一个程序可以跑成千上万个。

```go
// 启动goroutine（加go关键字）
go func() {
    fmt.Println("分身在修炼...")
}()

// 带参数的goroutine
go func(name string) {
    fmt.Printf("%s的分身在修炼\n", name)
}("韩道友")

// 匿名函数
go func() {
    time.Sleep(1 * time.Second)
    fmt.Println("修炼完成")
}()

// 等待goroutine完成
func main() {
    go func() {
        fmt.Println("分身1")
    }()
    go func() {
        fmt.Println("分身2")
    }()
    time.Sleep(1 * time.Second)  // 简单等待（不推荐）
}
```

**注意：** main函数退出，所有goroutine都会结束！

## 二、Channel - 传音入密

Channel是goroutine之间通信的管道，**"不要通过共享内存来通信，要通过通信来共享内存"**

### 1. 基本用法

```go
// 创建channel
ch := make(chan int)        // 无缓冲channel
ch := make(chan int, 10)    // 带缓冲channel

// 发送数据
ch <- 42

// 接收数据
value := <-ch

// 关闭channel（只发送方能关闭）
close(ch)

// 接收时判断是否关闭
value, ok := <-ch
if !ok {
    fmt.Println("channel已关闭")
}
```

### 2. 无缓冲vs带缓冲

```go
// 无缓冲channel（同步）
ch := make(chan int)
go func() {
    ch <- 42  // 会阻塞，直到有人接收
}()
value := <-ch  // 会阻塞，直到有人发送

// 带缓冲channel（异步）
ch := make(chan int, 3)
ch <- 1  // 不阻塞
ch <- 2  // 不阻塞
ch <- 3  // 不阻塞
// ch <- 4  // 阻塞，缓冲满了
```

### 3. 方向

```go
// 只发送
func sender(ch chan<- int) {
    ch <- 42
    // value := <-ch  // 编译错误
}

// 只接收
func receiver(ch <-chan int) {
    value := <-ch
    // ch <- 42  // 编译错误
}

// 双向
func both(ch chan int) {
    ch <- 42
    value := <-ch
}
```

### 4. select - 多路复用

```go
select {
case value := <-ch1:
    fmt.Println("从ch1收到:", value)
case value := <-ch2:
    fmt.Println("从ch2收到:", value)
case ch3 <- 42:
    fmt.Println("发送到ch3成功")
default:
    fmt.Println("没有channel就绪")
}
```

### 5. 超时控制

```go
select {
case result := <-ch:
    fmt.Println("收到结果:", result)
case <-time.After(3 * time.Second):
    fmt.Println("超时了！")
}
```

## 三、WaitGroup - 等待分身归来

```go
import "sync"

func main() {
    var wg sync.WaitGroup

    for i := 0; i < 5; i++ {
        wg.Add(1)  // 增加计数
        go func(id int) {
            defer wg.Done()  // 完成时减少计数
            fmt.Printf("分身%d修炼中...\n", id)
            time.Sleep(time.Second)
        }(i)
    }

    wg.Wait()  // 等待所有goroutine完成
    fmt.Println("所有分身修炼完成")
}
```

## 四、Mutex - 护体罡气

当多个goroutine访问共享数据时，需要加锁保护。

```go
import "sync"

type SafeCounter struct {
    mu    sync.Mutex
    count int
}

func (c *SafeCounter) Increment() {
    c.mu.Lock()         // 加锁
    defer c.mu.Unlock() // 解锁
    c.count++
}

func (c *SafeCounter) Value() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.count
}

// RWMutex - 读写锁
type SafeMap struct {
    mu   sync.RWMutex
    data map[string]int
}

func (m *SafeMap) Get(key string) int {
    m.mu.RLock()         // 读锁
    defer m.mu.RUnlock()
    return m.data[key]
}

func (m *SafeMap) Set(key string, value int) {
    m.mu.Lock()          // 写锁
    defer m.mu.Unlock()
    m.data[key] = value
}
```

## 五、Once - 只执行一次

```go
import "sync"

var (
    once sync.Once
    db   *Database
)

func GetDB() *Database {
    once.Do(func() {  // 只执行一次
        db = initDatabase()
    })
    return db
}
```

## 六、Context - 上下文控制

Context用于控制goroutine的生命周期、超时、取消。

```go
import "context"

// 1. 根context
ctx := context.Background()

// 2. 带取消的context
ctx, cancel := context.WithCancel(context.Background())
go func() {
    time.Sleep(2 * time.Second)
    cancel()  // 取消
}()

select {
case <-ctx.Done():
    fmt.Println("被取消:", ctx.Err())
case <-time.After(5 * time.Second):
    fmt.Println("正常完成")
}

// 3. 带超时的context
ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
defer cancel()

select {
case <-ctx.Done():
    fmt.Println("超时:", ctx.Err())
}

// 4. 带截止时间的context
deadline := time.Now().Add(5 * time.Second)
ctx, cancel := context.WithDeadline(context.Background(), deadline)
defer cancel()

// 5. 传值
ctx = context.WithValue(ctx, "userId", "12345")
userId := ctx.Value("userId").(string)
```

## 七、Worker Pool - 分身阵法

```go
func worker(id int, jobs <-chan int, results chan<- int) {
    for job := range jobs {
        fmt.Printf("分身%d处理任务%d\n", id, job)
        time.Sleep(time.Second)
        results <- job * 2
    }
}

func main() {
    jobs := make(chan int, 100)
    results := make(chan int, 100)

    // 启动3个worker
    for i := 1; i <= 3; i++ {
        go worker(i, jobs, results)
    }

    // 发送5个任务
    for j := 1; j <= 5; j++ {
        jobs <- j
    }
    close(jobs)

    // 收集结果
    for i := 1; i <= 5; i++ {
        <-results
    }
}
```
```go
package main

import (
    "fmt"
    "sync"
    "time"
)

type Task struct {
    ID     int
    Data   string
    Result string
}

func worker(id int, tasks <-chan Task, results chan<- Task, wg *sync.WaitGroup) {
    defer wg.Done()
    
    for task := range tasks {
        fmt.Printf("【分身%d】处理任务%d: %s\n", id, task.ID, task.Data)
        time.Sleep(time.Second) // 模拟处理
        
        // 处理结果
        task.Result = fmt.Sprintf("处理结果: %s的副本", task.Data)
        results <- task
    }
}

func main() {
    const numWorkers = 3
    const numTasks = 10
    
    tasks := make(chan Task, 100)
    results := make(chan Task, 100)
    
    // 使用 WaitGroup 等待所有 worker 完成
    var wg sync.WaitGroup
    
    // 启动 worker 池
    for i := 1; i <= numWorkers; i++ {
        wg.Add(1)
        go worker(i, tasks, results, &wg)
    }
    
    // 发送任务
    go func() {
        for i := 1; i <= numTasks; i++ {
            tasks <- Task{
                ID:   i,
                Data: fmt.Sprintf("任务数据%d", i),
            }
        }
        close(tasks) // 所有任务发送完毕，关闭任务通道
    }()
    
    // 等待所有 worker 完成，然后关闭结果通道
    go func() {
        wg.Wait()
        close(results)
    }()
    
    // 收集结果
    for result := range results {
        fmt.Printf("✅ 任务%d完成: %s\n", result.ID, result.Result)
    }
}
```

## 八、错误处理 - 并发版

```go
import (
    "errors"
    "golang.org/x/sync/errgroup"
)

// 使用errgroup
func main() {
    g, ctx := errgroup.WithContext(context.Background())

    g.Go(func() error {
        time.Sleep(1 * time.Second)
        fmt.Println("任务1")
        return nil
    })

    g.Go(func() error {
        time.Sleep(2 * time.Second)
        fmt.Println("任务2")
        return errors.New("任务2失败了")
    })

    g.Go(func() error {
        <-ctx.Done()  // 前面的任务失败，这个会被取消
        fmt.Println("任务3被取消")
        return ctx.Err()
    })

    if err := g.Wait(); err != nil {
        fmt.Println("有任务失败:", err)
    }
}
```

## 九、并发模式 - 阵法精要

### 1. Fan-out/Fan-in

```go
// Fan-out: 分发任务
func fanOut(ch <-chan int, workerCount int) []<-chan int {
    outs := make([]<-chan int, workerCount)
    for i := 0; i < workerCount; i++ {
        outs[i] = worker(ch)
    }
    return outs
}

// Fan-in: 合并结果
func fanIn(ins ...<-chan int) <-chan int {
    out := make(chan int)
    var wg sync.WaitGroup

    for _, in := range ins {
        wg.Add(1)
        go func(ch <-chan int) {
            defer wg.Done()
            for v := range ch {
                out <- v
            }
        }(in)
    }

    go func() {
        wg.Wait()
        close(out)
    }()

    return out
}
```

### 2. Pipeline

```go
// 阶段1: 生成数据
func generate(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        for _, n := range nums {
            out <- n
        }
        close(out)
    }()
    return out
}

// 阶段2: 处理数据
func square(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        for n := range in {
            out <- n * n
        }
        close(out)
    }()
    return out
}

// 使用
func main() {
    c := generate(1, 2, 3, 4, 5)
    out := square(c)

    for n := range out {
        fmt.Println(n)  // 1, 4, 9, 16, 25
    }
}
```

### 3. Rate Limiting - 限流

```go
// 令牌桶限流
func rateLimit() {
    requests := make(chan int, 5)
    for i := 1; i <= 5; i++ {
        requests <- i
    }
    close(requests)

    limiter := time.Tick(200 * time.Millisecond)

    for req := range requests {
        <-limiter  // 每200ms处理一个
        fmt.Println("请求", req, time.Now())
    }
}
```

## 十、并发安全 - 避免走火入魔

```go
// ❌ 危险：数据竞争
var counter int
func badIncrement() {
    counter++  // 多个goroutine同时修改会出问题
}

// ✅ 安全：使用Mutex
var (
    mu      sync.Mutex
    counter int
)
func safeIncrement() {
    mu.Lock()
    defer mu.Unlock()
    counter++
}

// ✅ 安全：使用atomic（适合简单操作）
import "sync/atomic"
var counter int64
func atomicIncrement() {
    atomic.AddInt64(&counter, 1)
}

// 检测数据竞争
// go run -race main.go
```

## 十一、最佳实践 - 修炼心得

1. **优先使用channel而不是共享内存**
2. **不要在goroutine中直接访问外部变量，要传参**
3. **记得关闭channel，但不要从接收方关闭**
4. **使用context控制goroutine生命周期**
5. **避免goroutine泄漏**
6. **用`go test -race`检测数据竞争**
7. **性能瓶颈优先考虑channel缓冲大小**
8. **错误处理要到位，goroutine中的错误要传出来**

---

**并发心法口诀：**

Goroutine分身千万，Channel传音入密
WaitGroup等待归来，Mutex护体罡气
Context控制生死，Select多路复用
Worker Pool阵法精妙，Pipeline流水作业

道友，并发之道可还记住了？多练多悟，方能大成！🌙
