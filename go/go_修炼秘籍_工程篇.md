# Go工程心法 - 宗门之道

## 一、项目结构 - 宗门布局

### 1. 标准项目结构

```
myproject/
├── cmd/                    # 主程序入口
│   ├── server/            # 服务端
│   │   └── main.go
│   └── client/            # 客户端
│       └── main.go
├── internal/               # 私有应用代码
│   ├── handler/           # HTTP处理器
│   ├── service/           # 业务逻辑
│   ├── repository/        # 数据访问
│   └── model/             # 数据模型
├── pkg/                    # 公共库（可被外部使用）
│   ├── utils/
│   └── middleware/
├── api/                    # API定义
│   ├── proto/             # protobuf文件
│   └── swagger/           # swagger文档
├── config/                 # 配置文件
│   ├── config.yaml
│   └── config.go
├── scripts/                # 脚本
│   ├── build.sh
│   └── deploy.sh
├── docs/                   # 文档
├── test/                   # 测试
├── go.mod
├── go.sum
├── Makefile
├── README.md
└── .gitignore
```

### 2. 简化版结构（小型项目）

```
myproject/
├── main.go                # 入口
├── config/                # 配置
├── models/                # 模型
├── handlers/              # 处理器
├── services/              # 服务
├── repositories/          # 仓储
├── utils/                 # 工具
├── middleware/            # 中间件
└── test/                  # 测试
```

## 二、依赖管理 - 丹方管理

### 1. go mod 命令

```bash
# 初始化模块
go mod init myproject

# 添加依赖
go get github.com/gin-gonic/gin

# 添加特定版本
go get github.com/gin-gonic/gin@v1.9.1

# 升级依赖
go get -u github.com/gin-gonic/gin

# 升级所有依赖
go get -u ./...

# 整理依赖
go mod tidy

# 下载依赖
go mod download

# 验证依赖
go mod verify

# 查看依赖图
go mod graph

# 查看为什么需要某个依赖
go mod why github.com/gin-gonic/gin

# 编辑依赖
go mod edit
```

### 2. go.sum 文件

go.sum记录所有依赖的校验和，确保依赖完整性。**不要手动编辑！**

### 3. 版本管理

```bash
# 查看可用版本
go list -m -versions github.com/gin-gonic/gin

# 切换版本
go get github.com/gin-gonic/gin@v1.8.0

# 回退到上一个版本
go get github.com/gin-gonic/gin@latest
```

## 三、测试之道 - 试炼场

### 1. 单元测试

```go
// calculator_test.go
package calculator

import "testing"

func TestAdd(t *testing.T) {
    tests := []struct {
        name     string
        a, b     int
        expected int
    }{
        {"正数", 2, 3, 5},
        {"负数", -2, -3, -5},
        {"零", 0, 0, 0},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := Add(tt.a, tt.b)
            if result != tt.expected {
                t.Errorf("Add(%d, %d) = %d; 期望 %d", tt.a, tt.b, result, tt.expected)
            }
        })
    }
}
```

### 2. Mock测试

```go
// 使用gomock
//go:generate mockgen -source=repository.go -destination=mocks/mock_repository.go

type UserRepository interface {
    FindByID(id int) (*User, error)
}

func TestUserService_GetUser(t *testing.T) {
    ctrl := gomock.NewController(t)
    defer ctrl.Finish()

    mockRepo := mocks.NewMockUserRepository(ctrl)
    mockRepo.EXPECT().FindByID(1).Return(&User{ID: 1, Name: "韩道友"}, nil)

    service := NewUserService(mockRepo)
    user, err := service.GetUser(1)

    if err != nil {
        t.Errorf("期望无错误，得到 %v", err)
    }
    if user.Name != "韩道友" {
        t.Errorf("期望名字是'韩道友'，得到 '%s'", user.Name)
    }
}
```

### 3. 集成测试

```go
// 测试数据库操作
func TestUserRepository_Create(t *testing.T) {
    db := setupTestDB(t)  // 设置测试数据库
    defer teardownTestDB(t, db)

    repo := NewUserRepository(db)
    user := &User{Name: "测试用户", Email: "test@example.com"}

    err := repo.Create(user)
    if err != nil {
        t.Fatalf("创建用户失败: %v", err)
    }

    // 验证
    found, err := repo.FindByID(user.ID)
    if err != nil {
        t.Fatalf("查询用户失败: %v", err)
    }
    if found.Name != "测试用户" {
        t.Errorf("期望名字是'测试用户'，得到 '%s'", found.Name)
    }
}
```

### 4. 基准测试

```go
func BenchmarkAdd(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Add(2, 3)
    }
}

func BenchmarkParallelAdd(b *testing.B) {
    b.RunParallel(func(pb *testing.PB) {
        for pb.Next() {
            Add(2, 3)
        }
    })
}
```

### 5. 测试覆盖率

```bash
# 运行测试并显示覆盖率
go test -cover ./...

# 生成覆盖率报告
go test -coverprofile=coverage.out ./...

# 查看覆盖率
go tool cover -html=coverage.out

# 设置覆盖率阈值
go test -coverprofile=coverage.out -covermode=atomic ./...
```

## 四、构建与部署 - 飞升仪式

### 1. Makefile

```makefile
.PHONY: build run test clean docker

# 变量
APP_NAME=myapp
VERSION=$(shell git describe --tags --always --dirty)
BUILD_TIME=$(shell date +%Y-%m-%d\ %H:%M:%S)

# 构建
build:
    go build -ldflags "-X main.Version=$(VERSION) -X main.BuildTime=$(BUILD_TIME)" -o bin/$(APP_NAME) cmd/server/main.go

# 运行
run:
    go run cmd/server/main.go

# 测试
test:
    go test -v ./...

# 测试覆盖率
test-coverage:
    go test -coverprofile=coverage.out ./...
    go tool cover -html=coverage.out

# 清理
clean:
    rm -rf bin/

# Docker构建
docker-build:
    docker build -t $(APP_NAME):$(VERSION) .

# Docker运行
docker-run:
    docker run -p 8080:8080 $(APP_NAME):$(VERSION)
```

### 2. Dockerfile

```dockerfile
# 多阶段构建
FROM golang:1.21-alpine AS builder

WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o server cmd/server/main.go

# 最终镜像
FROM alpine:latest

RUN apk --no-cache add ca-certificates
WORKDIR /root/

COPY --from=builder /app/server .
COPY config/config.yaml ./config/

EXPOSE 8080
CMD ["./server"]
```

### 3. docker-compose.yml

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8080:8080"
    depends_on:
      - db
      - redis
    environment:
      - DB_HOST=db
      - REDIS_HOST=redis

  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: password
      MYSQL_DATABASE: mydb
    volumes:
      - db_data:/var/lib/mysql

  redis:
    image: redis:7-alpine

volumes:
  db_data:
```

## 五、性能优化 - 灵力提升

### 1. pprof 性能分析

```go
import (
    _ "net/http/pprof"
    "net/http"
)

func main() {
    // 启动pprof
    go func() {
        http.ListenAndServe("localhost:6060", nil)
    }()

    // 你的应用代码
}
```

```bash
# CPU分析
go tool pprof http://localhost:6060/debug/pprof/profile

# 内存分析
go tool pprof http://localhost:6060/debug/pprof/heap

# Goroutine分析
go tool pprof http://localhost:6060/debug/pprof/goroutine

# 可视化
go tool pprof -http=:8080 http://localhost:6060/debug/pprof/profile
```

### 2. 优化技巧

```go
// 1. 预分配切片容量
// ❌ 差
var s []int
for i := 0; i < 1000; i++ {
    s = append(s, i)  // 多次扩容
}

// ✅ 好
s := make([]int, 0, 1000)  // 预分配
for i := 0; i < 1000; i++ {
    s = append(s, i)
}

// 2. 使用strings.Builder拼接字符串
// ❌ 差
s := ""
for i := 0; i < 1000; i++ {
    s += "hello"  // 每次都创建新字符串
}

// ✅ 好
var builder strings.Builder
for i := 0; i < 1000; i++ {
    builder.WriteString("hello")
}
s := builder.String()

// 3. 避免不必要的转换
// ❌ 差
n := 123
s := fmt.Sprintf("%d", n)  // 慢

// ✅ 好
s := strconv.Itoa(n)  // 快

// 4. 使用sync.Pool复用对象
var bufPool = sync.Pool{
    New: func() interface{} {
        return new(bytes.Buffer)
    },
}

func processData() {
    buf := bufPool.Get().(*bytes.Buffer)
    defer bufPool.Put(buf)
    buf.Reset()
    // 使用buf...
}

// 5. 减少内存分配
// ❌ 差
func sum(nums []int) int {
    total := 0
    for _, n := range nums {
        total += n
    }
    return total
}

// ✅ 好（避免闭包分配）
func sum(nums []int) int {
    total := 0
    for i := 0; i < len(nums); i++ {
        total += nums[i]
    }
    return total
}
```

### 3. 并发优化

```go
// 1. 使用worker pool
func processItems(items []Item) {
    workerCount := runtime.NumCPU()
    jobs := make(chan Item, len(items))
    results := make(chan Result, len(items))

    // 启动worker
    for i := 0; i < workerCount; i++ {
        go worker(jobs, results)
    }

    // 发送任务
    for _, item := range items {
        jobs <- item
    }
    close(jobs)

    // 收集结果
    for i := 0; i < len(items); i++ {
        <-results
    }
}

// 2. 使用context控制超时
func fetchData(ctx context.Context) error {
    ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
    defer cancel()

    select {
    case data := <-dataChan:
        process(data)
    case <-ctx.Done():
        return ctx.Err()
    }
}
```

## 六、错误处理 - 渡劫之道

### 1. 错误包装

```go
// Go 1.13+ 错误包装
func processFile(filename string) error {
    data, err := os.ReadFile(filename)
    if err != nil {
        return fmt.Errorf("读取文件 %s 失败: %w", filename, err)
    }
    // 处理data...
    return nil
}

// 检查错误类型
if err != nil {
    var pathErr *os.PathError
    if errors.As(err, &pathErr) {
        fmt.Printf("路径错误: %v\n", pathErr.Path)
    }
}

// 检查特定错误
if errors.Is(err, os.ErrNotExist) {
    fmt.Println("文件不存在")
}
```

### 2. 自定义错误

```go
// 定义错误类型
type AppError struct {
    Code    int
    Message string
    Err     error
}

func (e *AppError) Error() string {
    if e.Err != nil {
        return fmt.Sprintf("[%d] %s: %v", e.Code, e.Message, e.Err)
    }
    return fmt.Sprintf("[%d] %s", e.Code, e.Message)
}

func (e *AppError) Unwrap() error {
    return e.Err
}

// 使用
func processData() error {
    if someCondition {
        return &AppError{
            Code:    400,
            Message: "数据格式错误",
            Err:     errors.New("缺少必需字段"),
        }
    }
    return nil
}
```

## 七、日志与监控 - 天眼术

### 1. 结构化日志

```go
import (
    "go.uber.org/zap"
    "go.uber.org/zap/zapcore"
)

func initLogger() *zap.Logger {
    config := zap.NewProductionConfig()
    config.EncoderConfig.TimeKey = "timestamp"
    config.EncoderConfig.EncodeTime = zapcore.ISO8601TimeEncoder

    logger, _ := config.Build()
    return logger
}

func main() {
    logger := initLogger()
    defer logger.Sync()

    logger.Info("应用启动",
        zap.String("version", "1.0.0"),
        zap.Int("port", 8080),
    )

    logger.Error("处理失败",
        zap.String("user", "韩道友"),
        zap.Error(errors.New("数据库连接失败")),
    )
}
```

### 2. 指标监控

```go
import (
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

var (
    requestCount = prometheus.NewCounterVec(
        prometheus.CounterOpts{
            Name: "http_requests_total",
            Help: "Total number of HTTP requests",
        },
        []string{"method", "path"},
    )

    requestDuration = prometheus.NewHistogramVec(
        prometheus.HistogramOpts{
            Name:    "http_request_duration_seconds",
            Help:    "HTTP request duration",
            Buckets: prometheus.DefBuckets,
        },
        []string{"method", "path"},
    )
)

func init() {
    prometheus.MustRegister(requestCount)
    prometheus.MustRegister(requestDuration)
}

func middleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        start := time.Now()

        c.Next()

        requestCount.WithLabelValues(c.Request.Method, c.FullPath()).Inc()
        requestDuration.WithLabelValues(c.Request.Method, c.FullPath()).Observe(time.Since(start).Seconds())
    }
}

// 暴露指标
r.GET("/metrics", gin.WrapH(promhttp.Handler()))
```

## 八、代码规范 - 门规

### 1. 命名规范

```go
// 包名：小写，简短，有意义
package user  // ✅ 好
package usermanager  // ❌ 太长

// 常量：大写，驼峰
const MaxRetries = 3
const DefaultTimeout = 30

// 变量：驼峰
var userName string  // ✅ 导出
var userID int       // ✅ 导出
var internalVar int  // ✅ 私有

// 函数：驼峰，动词开头
func GetUser() {}      // ✅ 导出
func getUser() {}      // ✅ 私有
func calculateSum() {} // ✅ 私有

// 接口：通常以 -er 结尾
type Reader interface {}
type Writer interface {}
type Stringer interface {}
```

### 2. 注释规范

```go
// Package user 提供用户管理功能
package user

// User 表示一个用户实体
type User struct {
    ID    uint   `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
}

// CreateUser 创建一个新用户
// 如果邮箱已存在，返回错误
func CreateUser(name, email string) (*User, error) {
    // 实现...
}
```

### 3. 错误处理

```go
// ✅ 好：及时处理错误
data, err := os.ReadFile("file.txt")
if err != nil {
    return fmt.Errorf("读取文件失败: %w", err)
}

// ❌ 差：忽略错误
data, _ := os.ReadFile("file.txt")

// ✅ 好：包装错误
if err := process(); err != nil {
    return fmt.Errorf("处理失败: %w", err)
}
```

---

**工程心法口诀：**
项目结构要清晰，依赖管理用go mod
测试覆盖要充分，构建部署自动化
性能分析用pprof，错误处理要规范
日志监控不能少，代码规范要遵守

道友，工程之道可还明白？💪
