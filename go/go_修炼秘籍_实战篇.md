# Go语言实战心法

## 一、Web服务 - 开宗立派

### 1. RESTful API（Gin框架）

```go
package main

import (
    "net/http"
    "github.com/gin-gonic/gin"
)

// 修仙者实体
type Cultivator struct {
    ID    uint   `json:"id" gorm:"primaryKey"`
    Name  string `json:"name" binding:"required"`
    Level string `json:"level"`
    Power int    `json:"power"`
}

func main() {
    r := gin.Default()

    // 路由分组
    api := r.Group("/api/v1")
    {
        api.GET("/cultivators", listCultivators)
        api.GET("/cultivators/:id", getCultivator)
        api.POST("/cultivators", createCultivator)
        api.PUT("/cultivators/:id", updateCultivator)
        api.DELETE("/cultivators/:id", deleteCultivator)
    }

    r.Run(":8080") // 启动服务
}

// 列表查询
func listCultivators(c *gin.Context) {
    // 分页参数
    page := c.DefaultQuery("page", "1")
    pageSize := c.DefaultQuery("pageSize", "10")
    
    // 筛选参数
    level := c.Query("level")
    
    // TODO: 数据库查询
    cultivators := []Cultivator{
        {ID: 1, Name: "韩道友", Level: "金丹", Power: 9999},
    }
    
    c.JSON(http.StatusOK, gin.H{
        "code": 0,
        "data": cultivators,
        "meta": gin.H{
            "page": page,
            "pageSize": pageSize,
        },
    })
}

// 详情查询
func getCultivator(c *gin.Context) {
    id := c.Param("id")
    
    // TODO: 数据库查询
    cultivator := Cultivator{
        ID: 1, Name: "韩道友", Level: "金丹", Power: 9999,
    }
    
    c.JSON(http.StatusOK, gin.H{
        "code": 0,
        "data": cultivator,
    })
}

// 创建
func createCultivator(c *gin.Context) {
    var req Cultivator
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{
            "code": 400,
            "msg":  "参数错误: " + err.Error(),
        })
        return
    }
    
    // TODO: 数据库保存
    
    c.JSON(http.StatusCreated, gin.H{
        "code": 0,
        "msg":  "创建成功",
        "data": req,
    })
}

// 更新
func updateCultivator(c *gin.Context) {
    id := c.Param("id")
    var req Cultivator
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"code": 400, "msg": err.Error()})
        return
    }
    
    // TODO: 数据库更新
    
    c.JSON(http.StatusOK, gin.H{"code": 0, "msg": "更新成功"})
}

// 删除
func deleteCultivator(c *gin.Context) {
    id := c.Param("id")
    
    // TODO: 数据库删除
    
    c.JSON(http.StatusOK, gin.H{"code": 0, "msg": "删除成功"})
}
```

### 2. 中间件

```go
// 日志中间件
func Logger() gin.HandlerFunc {
    return func(c *gin.Context) {
        start := time.Now()
        path := c.Request.URL.Path
        
        c.Next()  // 处理请求
        
        latency := time.Since(start)
        statusCode := c.Writer.Status()
        
        log.Printf("[%s] %s %d %v",
            c.Request.Method, path, statusCode, latency)
    }
}

// 认证中间件
func Auth() gin.HandlerFunc {
    return func(c *gin.Context) {
        token := c.GetHeader("Authorization")
        if token == "" {
            c.JSON(http.StatusUnauthorized, gin.H{
                "code": 401,
                "msg":  "未登录",
            })
            c.Abort()
            return
        }
        
        // 验证token
        userID, err := validateToken(token)
        if err != nil {
            c.JSON(http.StatusUnauthorized, gin.H{
                "code": 401,
                "msg":  "token无效",
            })
            c.Abort()
            return
        }
        
        c.Set("userID", userID)
        c.Next()
    }
}

// 使用中间件
func main() {
    r := gin.Default()
    r.Use(Logger())
    
    // 需要认证的路由
    auth := r.Group("/api")
    auth.Use(Auth())
    {
        auth.GET("/profile", getProfile)
    }
}

// CORS跨域中间件
func CORS() gin.HandlerFunc {
    return func(c *gin.Context) {
        c.Header("Access-Control-Allow-Origin", "*")
        c.Header("Access-Control-Allow-Methods", "GET,POST,PUT,DELETE,OPTIONS")
        c.Header("Access-Control-Allow-Headers", "Content-Type,Authorization")
        
        if c.Request.Method == "OPTIONS" {
            c.AbortWithStatus(http.StatusNoContent)
            return
        }
        
        c.Next()
    }
}
```

### 3. 配置管理

```go
// config/config.go
package config

import (
    "github.com/spf13/viper"
)

type Config struct {
    Server   ServerConfig
    Database DatabaseConfig
    Redis    RedisConfig
}

type ServerConfig struct {
    Port int
    Mode string
}

type DatabaseConfig struct {
    Host     string
    Port     int
    User     string
    Password string
    DBName   string
}

type RedisConfig struct {
    Addr     string
    Password string
    DB       int
}

func Load() (*Config, error) {
    viper.SetConfigName("config")
    viper.SetConfigType("yaml")
    viper.AddConfigPath(".")
    viper.AddConfigPath("./config")
    
    // 环境变量
    viper.AutomaticEnv()
    
    if err := viper.ReadInConfig(); err != nil {
        return nil, err
    }
    
    var cfg Config
    if err := viper.Unmarshal(&cfg); err != nil {
        return nil, err
    }
    
    return &cfg, nil
}

// config.yaml
/*
server:
  port: 8080
  mode: debug

database:
  host: localhost
  port: 3306
  user: root
  password: secret
  dbname: cultivation

redis:
  addr: localhost:6379
  password: ""
  db: 0
*/
```

## 二、数据库操作 - 藏经阁管理

### 1. Gorm基础

```go
package database

import (
    "fmt"
    "gorm.io/gorm"
    "gorm.io/driver/mysql"
)

var DB *gorm.DB

func Init(dsn string) error {
    var err error
    DB, err = gorm.Open(mysql.Open(dsn), &gorm.Config{})
    if err != nil {
        return err
    }
    
    // 自动迁移
    DB.AutoMigrate(&Cultivator{}, &Sect{})
    
    return nil
}

// 实体定义
type Cultivator struct {
    gorm.Model
    Name   string `gorm:"size:100;not null;uniqueIndex"`
    Level  string `gorm:"size:50;default:'筑基'"`
    Power  int    `gorm:"default:100"`
    SectID uint
    Sect   Sect
}

type Sect struct {
    gorm.Model
    Name        string `gorm:"size:100;not null;unique"`
    Description string
    Members     []Cultivator
}

// CRUD操作
func CreateCultivator(c *Cultivator) error {
    return DB.Create(c).Error
}

func GetCultivatorByID(id uint) (*Cultivator, error) {
    var c Cultivator
    err := DB.Preload("Sect").First(&c, id).Error
    return &c, err
}

func ListCultivators(page, pageSize int, level string) ([]Cultivator, int64, error) {
    var list []Cultivator
    var total int64
    
    query := DB.Model(&Cultivator{})
    if level != "" {
        query = query.Where("level = ?", level)
    }
    
    query.Count(&total)
    
    offset := (page - 1) * pageSize
    err := query.Offset(offset).Limit(pageSize).Find(&list).Error
    
    return list, total, err
}

func UpdateCultivator(id uint, updates map[string]interface{}) error {
    return DB.Model(&Cultivator{}).Where("id = ?", id).Updates(updates).Error
}

func DeleteCultivator(id uint) error {
    return DB.Delete(&Cultivator{}, id).Error
}

// 事务
func TransferSect(cultivatorID, newSectID uint) error {
    return DB.Transaction(func(tx *gorm.DB) error {
        // 更新修仙者宗门
        if err := tx.Model(&Cultivator{}).
            Where("id = ?", cultivatorID).
            Update("sect_id", newSectID).Error; err != nil {
            return err
        }
        
        // 更新原宗门成员数
        // ...
        
        // 更新新宗门成员数
        // ...
        
        return nil
    })
}
```

### 2. 复杂查询

```go
// 关联查询
func GetSectWithMembers(sectID uint) (*Sect, error) {
    var sect Sect
    err := DB.Preload("Members", "power > ?", 1000).
        First(&sect, sectID).Error
    return &sect, err
}

// 聚合查询
func GetCultivatorStats() (map[string]interface{}, error) {
    var total int64
    var avgPower float64
    var levelCounts []struct {
        Level string
        Count int
    }
    
    DB.Model(&Cultivator{}).Count(&total)
    DB.Model(&Cultivator{}).Select("AVG(power)").Scan(&avgPower)
    DB.Model(&Cultivator{}).
        Select("level, count(*) as count").
        Group("level").
        Scan(&levelCounts)
    
    return gin.H{
        "total":    total,
        "avgPower": avgPower,
        "byLevel":  levelCounts,
    }, nil
}

// 分组统计
func GetTopCultivators(limit int) ([]Cultivator, error) {
    var list []Cultivator
    err := DB.Order("power DESC").
        Limit(limit).
        Find(&list).Error
    return list, err
}

// 存在性检查
func NameExists(name string) (bool, error) {
    var count int64
    err := DB.Model(&Cultivator{}).
        Where("name = ?", name).
        Count(&count).Error
    return count > 0, err
}
```

## 三、微服务 - 分宗立派

### 1. 项目结构

```
cultivation-service/
├── cmd/
│   └── server/
│       └── main.go          # 入口文件
├── internal/
│   ├── handler/             # HTTP处理器
│   │   └── cultivator.go
│   ├── service/             # 业务逻辑
│   │   └── cultivator.go
│   ├── repository/          # 数据访问
│   │   └── cultivator.go
│   └── model/               # 实体定义
│       └── cultivator.go
├── pkg/                     # 公共包
│   ├── config/
│   ├── logger/
│   └── response/
├── api/
│   └── proto/               # Protobuf定义
├── configs/
│   └── config.yaml
├── scripts/
│   └── migrate.sql
├── go.mod
├── go.sum
├── Dockerfile
└── docker-compose.yaml
```

### 2. gRPC服务

```go
// api/proto/cultivator.proto
syntax = "proto3";

package cultivator;

option go_package = "github.com/example/cultivation-service/api/proto;proto";

service CultivatorService {
    rpc GetCultivator(GetRequest) returns (Cultivator);
    rpc ListCultivators(ListRequest) returns (ListResponse);
    rpc CreateCultivator(CreateRequest) returns (Cultivator);
}

message Cultivator {
    uint32 id = 1;
    string name = 2;
    string level = 3;
    int32 power = 4;
}

message GetRequest {
    uint32 id = 1;
}

message ListRequest {
    int32 page = 1;
    int32 page_size = 2;
    string level = 3;
}

message ListResponse {
    repeated Cultivator items = 1;
    int64 total = 2;
}

message CreateRequest {
    string name = 1;
    string level = 2;
}

// 生成代码
// protoc --go_out=. --go-grpc_out=. api/proto/cultivator.proto
```

```go
// internal/service/cultivator.go
package service

import (
    "context"
    pb "github.com/example/cultivation-service/api/proto"
)

type CultivatorService struct {
    pb.UnimplementedCultivatorServiceServer
    repo Repository
}

func NewCultivatorService(repo Repository) *CultivatorService {
    return &CultivatorService{repo: repo}
}

func (s *CultivatorService) GetCultivator(ctx context.Context, req *pb.GetRequest) (*pb.Cultivator, error) {
    c, err := s.repo.GetByID(ctx, uint(req.Id))
    if err != nil {
        return nil, err
    }
    
    return &pb.Cultivator{
        Id:    uint32(c.ID),
        Name:  c.Name,
        Level: c.Level,
        Power: int32(c.Power),
    }, nil
}

func (s *CultivatorService) ListCultivators(ctx context.Context, req *pb.ListRequest) (*pb.ListResponse, error) {
    list, total, err := s.repo.List(ctx, int(req.Page), int(req.PageSize), req.Level)
    if err != nil {
        return nil, err
    }
    
    items := make([]*pb.Cultivator, len(list))
    for i, c := range list {
        items[i] = &pb.Cultivator{
            Id:    uint32(c.ID),
            Name:  c.Name,
            Level: c.Level,
            Power: int32(c.Power),
        }
    }
    
    return &pb.ListResponse{
        Items: items,
        Total: total,
    }, nil
}
```

```go
// cmd/server/main.go
package main

import (
    "log"
    "net"
    "google.golang.org/grpc"
    pb "github.com/example/cultivation-service/api/proto"
    "github.com/example/cultivation-service/internal/service"
)

func main() {
    // 初始化数据库
    // db := database.Init(cfg.Database)
    
    // 创建服务
    // repo := repository.NewCultivatorRepository(db)
    // svc := service.NewCultivatorService(repo)
    
    // 启动gRPC服务
    lis, err := net.Listen("tcp", ":50051")
    if err != nil {
        log.Fatal(err)
    }
    
    grpcServer := grpc.NewServer()
    pb.RegisterCultivatorServiceServer(grpcServer, svc)
    
    log.Println("gRPC服务启动: :50051")
    if err := grpcServer.Serve(lis); err != nil {
        log.Fatal(err)
    }
}
```

### 3. 服务发现与注册

```go
// 使用Consul
import (
    "github.com/hashicorp/consul/api"
)

func RegisterService(consulAddr, serviceName, serviceID, serviceAddr string, servicePort int) error {
    config := api.DefaultConfig()
    config.Address = consulAddr
    client, err := api.NewClient(config)
    if err != nil {
        return err
    }
    
    registration := &api.AgentServiceRegistration{
        ID:      serviceID,
        Name:    serviceName,
        Address: serviceAddr,
        Port:    servicePort,
        Check: &api.AgentServiceCheck{
            HTTP:     fmt.Sprintf("http://%s:%d/health", serviceAddr, servicePort),
            Interval: "10s",
            Timeout:  "1s",
        },
    }
    
    return client.Agent().ServiceRegister(registration)
}

func DiscoverService(consulAddr, serviceName string) (string, int, error) {
    config := api.DefaultConfig()
    config.Address = consulAddr
    client, err := api.NewClient(config)
    if err != nil {
        return "", 0, err
    }
    
    services, _, err := client.Health().Service(serviceName, "", true, nil)
    if err != nil {
        return "", 0, err
    }
    
    if len(services) == 0 {
        return "", 0, fmt.Errorf("服务未找到: %s", serviceName)
    }
    
    service := services[0].Service
    return service.Address, service.Port, nil
}
```

## 四、CLI工具 - 法器打造

### 1. Cobra框架

```go
// cmd/root.go
package cmd

import (
    "fmt"
    "os"
    "github.com/spf13/cobra"
)

var rootCmd = &cobra.Command{
    Use:   "cultivation",
    Short: "修仙之路CLI工具",
    Long:  "修仙者管理系统命令行工具",
}

func Execute() {
    if err := rootCmd.Execute(); err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
}

// cmd/cultivator.go
package cmd

import (
    "fmt"
    "github.com/spf13/cobra"
)

var cultivatorCmd = &cobra.Command{
    Use:   "cultivator",
    Short: "修仙者管理",
}

var listCmd = &cobra.Command{
    Use:   "list",
    Short: "列出所有修仙者",
    Run: func(cmd *cobra.Command, args []string) {
        level, _ := cmd.Flags().GetString("level")
        page, _ := cmd.Flags().GetInt("page")
        
        // 调用API
        fmt.Printf("列出修仙者: level=%s, page=%d\n", level, page)
    },
}

var createCmd = &cobra.Command{
    Use:   "create [name]",
    Short: "创建修仙者",
    Args:  cobra.ExactArgs(1),
    Run: func(cmd *cobra.Command, args []string) {
        name := args[0]
        level, _ := cmd.Flags().GetString("level")
        
        fmt.Printf("创建修仙者: name=%s, level=%s\n", name, level)
    },
}

func init() {
    rootCmd.AddCommand(cultivatorCmd)
    cultivatorCmd.AddCommand(listCmd, createCmd)
    
    listCmd.Flags().StringP("level", "l", "", "境界筛选")
    listCmd.Flags().IntP("page", "p", 1, "页码")
    
    createCmd.Flags().StringP("level", "l", "筑基", "初始境界")
}

// main.go
package main

import "cultivation/cmd"

func main() {
    cmd.Execute()
}
```

### 2. 交互式CLI

```go
package main

import (
    "fmt"
    "github.com/AlecAivazis/survey/v2"
)

func main() {
    // 单选
    var level string
    prompt := &survey.Select{
        Message: "选择境界:",
        Options: []string{"筑基", "金丹", "元婴", "化神"},
    }
    survey.AskOne(prompt, &level)
    
    // 多选
    var skills []string
    prompt2 := &survey.MultiSelect{
        Message: "选择功法:",
        Options: []string{"炼丹", "炼器", "阵法", "符箓"},
    }
    survey.AskOne(prompt2, &skills)
    
    // 输入
    var name string
    prompt3 := &survey.Input{
        Message: "道号:",
    }
    survey.AskOne(prompt3, &name)
    
    // 确认
    var confirm bool
    prompt4 := &survey.Confirm{
        Message: "确认创建?",
    }
    survey.AskOne(prompt4, &confirm)
    
    // 密码输入
    var password string
    prompt5 := &survey.Password{
        Message: "输入口令:",
    }
    survey.AskOne(prompt5, &password)
    
    fmt.Printf("创建: %s, %s, %v\n", name, level, skills)
}
```

### 3. 进度显示

```go
package main

import (
    "time"
    "github.com/schollz/progressbar/v3"
)

func main() {
    // 进度条
    bar := progressbar.Default(100)
    for i := 0; i < 100; i++ {
        bar.Add(1)
        time.Sleep(50 * time.Millisecond)
    }
    
    // 带描述的进度条
    bar2 := progressbar.NewOptions(100,
        progressbar.OptionSetDescription("修炼中..."),
        progressbar.OptionSetWriter(os.Stderr),
        progressbar.OptionShowCount(),
        progressbar.OptionShowIts(),
        progressbar.OptionSetItsString("周天"),
    )
    
    for i := 0; i < 100; i++ {
        bar2.Add(1)
        time.Sleep(100 * time.Millisecond)
    }
}
```

## 五、部署 - 飞升上线

### 1. Docker部署

```dockerfile
# Dockerfile
# 构建阶段
FROM golang:1.21-alpine AS builder

WORKDIR /app

# 依赖缓存
COPY go.mod go.sum ./
RUN go mod download

# 编译
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main ./cmd/server

# 运行阶段
FROM alpine:latest

RUN apk --no-cache add ca-certificates tzdata

WORKDIR /root/

COPY --from=builder /app/main .
COPY --from=builder /app/configs ./configs

ENV TZ=Asia/Shanghai

EXPOSE 8080

CMD ["./main"]
```

```yaml
# docker-compose.yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - DB_HOST=db
      - DB_PORT=3306
      - DB_USER=root
      - DB_PASSWORD=secret
      - DB_NAME=cultivation
      - REDIS_ADDR=redis:6379
    depends_on:
      - db
      - redis
    restart: always

  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: cultivation
    volumes:
      - mysql_data:/var/lib/mysql
    ports:
      - "3306:3306"

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  mysql_data:
```

### 2. Kubernetes部署

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cultivation-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: cultivation
  template:
    metadata:
      labels:
        app: cultivation
    spec:
      containers:
      - name: cultivation
        image: cultivation-service:latest
        ports:
        - containerPort: 8080
        env:
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: cultivation-config
              key: db_host
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: cultivation-secret
              key: db_password
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: cultivation-service
spec:
  selector:
    app: cultivation
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: cultivation-ingress
spec:
  rules:
  - host: cultivation.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: cultivation-service
            port:
              number: 80
```

### 3. 健康检查

```go
// internal/handler/health.go
package handler

import (
    "net/http"
    "github.com/gin-gonic/gin"
)

func HealthCheck(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{
        "status": "ok",
    })
}

func ReadyCheck(db *gorm.DB, redis *redis.Client) gin.HandlerFunc {
    return func(c *gin.Context) {
        // 检查数据库
        sqlDB, err := db.DB()
        if err != nil {
            c.JSON(http.StatusServiceUnavailable, gin.H{
                "status": "database error",
            })
            return
        }
        
        if err := sqlDB.Ping(); err != nil {
            c.JSON(http.StatusServiceUnavailable, gin.H{
                "status": "database unreachable",
            })
            return
        }
        
        // 检查Redis
        if err := redis.Ping(context.Background()).Err(); err != nil {
            c.JSON(http.StatusServiceUnavailable, gin.H{
                "status": "redis unreachable",
            })
            return
        }
        
        c.JSON(http.StatusOK, gin.H{
            "status": "ready",
        })
    }
}

// 使用
func main() {
    r := gin.Default()
    r.GET("/health", HealthCheck)
    r.GET("/ready", ReadyCheck(db, redis))
}
```

---

**实战要诀：**
- 项目结构要清晰，分层架构是根本
- 配置外置，日志规范，健康检查不可少
- 中间件合理使用，认证授权要分清
- Docker化部署，K8s编排，CI/CD自动化
- 监控告警，日志收集，问题排查快准狠

道友，实战篇记住了吗？纸上得来终觉浅，绝知此事要躬行！🌙
