# Go语言筑基心法

## 一、基础语法 - 根基要稳

### 1. 变量声明

```go
// 标准声明
var name string = "韩道友"
var age int = 25

// 短变量声明（函数内常用）
name := "韩道友"
age := 25

// 多变量声明
var (
    name string = "韩道友"
    age  int    = 25
)

// 批量短声明
name, age := "韩道友", 25
```

### 2. 数据类型

```go
// 基本类型
var b bool = true
var i int = 42
var f float64 = 3.14
var s string = "修仙"

// 数组（固定长度）
var arr [5]int
arr := [5]int{1, 2, 3, 4, 5}

// 切片（动态长度）
slice := []int{1, 2, 3}
slice = append(slice, 4)  // 追加元素

// 映射（字典）
m := make(map[string]int)
m["道友"] = 100
m["境界"] = 999

// 指针
x := 42
p := &x  // p 是 x 的地址
*p = 99  // 通过指针修改 x
```

### 3. 流程控制

```go
// if-else
if age < 18 {
    fmt.Println("筑基期")
} else if age < 30 {
    fmt.Println("金丹期")
} else {
    fmt.Println("元婴期")
}

// switch
switch level {
case "筑基":
    fmt.Println("刚入门")
case "金丹":
    fmt.Println("小有成就")
default:
    fmt.Println("高深莫测")
}

// for循环（Go只有for）
// 方式1：类似while
i := 0
for i < 10 {
    fmt.Println(i)
    i++
}

// 方式2：标准for
for i := 0; i < 10; i++ {
    fmt.Println(i)
}

// 方式3：遍历切片/映射
for idx, val := range slice {
    fmt.Println(idx, val)
}
```

## 二、函数 - 灵力运转

```go
// 基本函数
func greet(name string) {
    fmt.Printf("道友%s，幸会！\n", name)
}

// 多返回值（Go特色）
func divide(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("不能除以零")
    }
    return a / b, nil
}

// 命名返回值
func calc(a, b int) (sum, diff int) {
    sum = a + b
    diff = a - b
    return  // 直接返回
}

// 可变参数
func sum(nums ...int) int {
    total := 0
    for _, n := range nums {
        total += n
    }
    return total
}

// 闭包
func makeAdder(x int) func(int) int {
    return func(y int) int {
        return x + y
    }
}
```

## 三、结构体与方法 - 塑造法相

```go
// 结构体定义
type Daoist struct {
    Name   string
    Level  string
    Power  int
}

// 构造函数
func NewDaoist(name string) *Daoist {
    return &Daoist{
        Name:   name,
        Level:  "筑基期",
        Power:  100,
    }
}

// 方法（值接收者）
func (d Daoist) Greet() {
    fmt.Printf("%s道友，修为：%s\n", d.Name, d.Level)
}

// 方法（指针接收者，可修改）
func (d *Daoist) Upgrade() {
    d.Level = "金丹期"
    d.Power += 100
}

// 接口
type Cultivator interface {
    Greet()
    Upgrade()
}

// 任意类型实现接口
func (d *Daoist) Cultivate() {
    d.Power += 10
}
```

## 四、接口 - 万法归一

```go
// 接口定义
type Speaker interface {
    Speak() string
}

type Dog struct{}
type Cat struct{}

func (d Dog) Speak() string { return "汪汪" }
func (c Cat) Speak() string { return "喵喵" }

// 接口使用
func MakeSound(s Speaker) {
    fmt.Println(s.Speak())
}

// 空接口（可存任意类型）
var anything interface{} = 42
anything = "修仙"
anything = []int{1, 2, 3}

// 类型断言
if v, ok := anything.(int); ok {
    fmt.Println("是整数:", v)
}

// 类型switch
switch v := anything.(type) {
case int:
    fmt.Println("整数:", v)
case string:
    fmt.Println("字符串:", v)
default:
    fmt.Println("其他类型")
}
```

## 五、错误处理 - 渡劫心法

```go
// Go的错误是值，不是异常
import "errors"

// 创建错误
err := errors.New("修炼出错了")

// fmt.Errorf带格式化
err := fmt.Errorf("境界突破失败: %v", reason)

// 自定义错误类型
type CultivationError struct {
    Level string
    Reason string
}

func (e *CultivationError) Error() string {
    return fmt.Sprintf("%s突破失败: %s", e.Level, e.Reason)
}

// 错误处理
func breakthrough() error {
    if power < 1000 {
        return &CultivationError{
            Level: "金丹",
            Reason: "灵力不足",
        }
    }
    return nil
}

// 调用
if err := breakthrough(); err != nil {
    // 处理错误
    if ce, ok := err.(*CultivationError); ok {
        fmt.Printf("自定义错误: %v\n", ce)
    } else {
        fmt.Printf("普通错误: %v\n", err)
    }
}
```

## 六、包管理 - 丹方收集

```bash
# 初始化模块
go mod init myproject

# 添加依赖
go get github.com/gin-gonic/gin

# 升级依赖
go get -u github.com/gin-gonic/gin

# 整理依赖
go mod tidy

# 下载依赖
go mod download

# 查看依赖图
go mod graph
```

## 七、常用包 - 法器库

```go
import (
    "fmt"      // 格式化I/O
    "strings"  // 字符串操作
    "strconv"  // 字符串转换
    "math"     // 数学运算
    "time"     // 时间处理
    "os"       // 操作系统接口
    "io"       // I/O基础接口
    "bufio"    // 缓冲I/O
    "encoding/json"  // JSON处理
)

// strings示例
s := "hello world"
fmt.Println(strings.ToUpper(s))        // HELLO WORLD
fmt.Println(strings.Contains(s, "ll"))  // true
fmt.Println(strings.Split(s, " "))     // [hello world]

// time示例
now := time.Now()
fmt.Println(now.Format("2006-01-02 15:04:05"))
later := now.Add(24 * time.Hour)

// JSON示例
type Daoist struct {
    Name  string `json:"name"`
    Level string `json:"level"`
}
d := Daoist{Name: "韩道友", Level: "金丹"}
data, _ := json.Marshal(d)
fmt.Println(string(data))  // {"name":"韩道友","level":"金丹"}
```

## 八、测试 - 试炼之道

```go
// 文件名: daoist_test.go
package main

import "testing"

func TestAdd(t *testing.T) {
    result := add(2, 3)
    if result != 5 {
        t.Errorf("期望5，得到%d", result)
    }
}

// 表驱动测试
func TestAddTable(t *testing.T) {
    tests := []struct {
        a, b, expected int
    }{
        {1, 2, 3},
        {2, 3, 5},
        {-1, 1, 0},
    }

    for _, tt := range tests {
        result := add(tt.a, tt.b)
        if result != tt.expected {
            t.Errorf("%d + %d = %d; 期望%d", tt.a, tt.b, result, tt.expected)
        }
    }
}

// 基准测试
func BenchmarkAdd(b *testing.B) {
    for i := 0; i < b.N; i++ {
        add(2, 3)
    }
}
```

---

**筑基要诀：**
- 多写代码，少看视频
- 错误处理要习惯，别想着try-catch
- 接口是Go的灵魂，理解duck typing
- 测试要写，不然渡劫容易翻车

道友，筑基篇记住了吗？接下来就是并发心法了！🌙
