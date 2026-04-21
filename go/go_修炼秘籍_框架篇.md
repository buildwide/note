# Go框架心法 - 法器篇

## 一、Gin - Web框架之王

Gin是最流行的Go Web框架，性能强悍，API友好。

### 1. 快速上手

```go
package main

import (
    "github.com/gin-gonic/gin"
)

func main() {
    // 创建路由
    r := gin.Default()

    // 定义路由
    r.GET("/ping", func(c *gin.Context) {
        c.JSON(200, gin.H{
            "message": "pong",
        })
    })

    // 启动服务
    r.Run(":8080")  // 默认监听0.0.0.0:8080
}
```

### 2. 路由定义

```go
// GET请求
r.GET("/users", getUsers)
r.GET("/users/:id", getUser)

// POST请求
r.POST("/users", createUser)

// PUT/PATCH
r.PUT("/users/:id", updateUser)
r.PATCH("/users/:id", patchUser)

// DELETE
r.DELETE("/users/:id", deleteUser)

// 任意方法
r.Any("/test", func(c *gin.Context) {
    c.String(200, "任意方法")
})

// 路由分组
v1 := r.Group("/api/v1")
{
    v1.GET("/users", getUsers)
    v1.POST("/users", createUser)
}

v2 := r.Group("/api/v2")
v2.Use(AuthMiddleware())  // 分组中间件
{
    v2.GET("/users", getUsersV2)
}
```

### 3. 参数获取

```go
// 路径参数
r.GET("/users/:id", func(c *gin.Context) {
    id := c.Param("id")  // string类型
    c.JSON(200, gin.H{"id": id})
})

// 查询参数 ?name=韩道友&level=金丹
r.GET("/search", func(c *gin.Context) {
    name := c.Query("name")
    level := c.DefaultQuery("level", "筑基")  // 默认值
    c.JSON(200, gin.H{
        "name":  name,
        "level": level,
    })
})

// 表单参数
r.POST("/form", func(c *gin.Context) {
    name := c.PostForm("name")
    age := c.DefaultPostForm("age", "18")
    c.JSON(200, gin.H{
        "name": name,
        "age":  age,
    })
})

// JSON参数
type User struct {
    Name  string `json:"name"`
    Email string `json:"email"`
}

r.POST("/json", func(c *gin.Context) {
    var user User
    if err := c.ShouldBindJSON(&user); err != nil {
        c.JSON(400, gin.H{"error": err.Error()})
        return
    }
    c.JSON(200, user)
})

// 文件上传
r.POST("/upload", func(c *gin.Context) {
    file, err := c.FormFile("file")
    if err != nil {
        c.JSON(400, gin.H{"error": err.Error()})
        return
    }
    c.SaveUploadedFile(file, "./uploads/"+file.Filename)
    c.JSON(200, gin.H{"message": "上传成功"})
})
```

### 4. 响应返回

```go
// JSON响应
c.JSON(200, gin.H{"message": "成功"})

// 字符串响应
c.String(200, "纯文本")

// HTML响应
c.HTML(200, "index.html", gin.H{"title": "首页"})

// XML响应
c.XML(200, gin.H{"message": "XML"})

// 重定向
c.Redirect(302, "/login")

// 文件下载
c.File("./files/report.pdf")

// 设置header
c.Header("X-Custom-Header", "value")

// 设置cookie
c.SetCookie("name", "value", 3600, "/", "localhost", false, true)
```

### 5. 中间件

```go
// 自定义中间件
func Logger() gin.HandlerFunc {
    return func(c *gin.Context) {
        fmt.Printf("请求: %s %s\n", c.Request.Method, c.Request.URL.Path)
        c.Next()  // 继续处理
        fmt.Printf("响应状态: %d\n", c.Writer.Status())
    }
}

func Auth() gin.HandlerFunc {
    return func(c *gin.Context) {
        token := c.GetHeader("Authorization")
        if token == "" {
            c.JSON(401, gin.H{"error": "未授权"})
            c.Abort()  // 终止请求
            return
        }
        c.Next()
    }
}

// 全局中间件
r.Use(Logger())
r.Use(Auth())

// 路由中间件
r.GET("/protected", Auth(), func(c *gin.Context) {
    c.JSON(200, gin.H{"message": "受保护的路由"})
})

// 中间件链
r.GET("/chain", Auth(), Logger(), func(c *gin.Context) {
    c.JSON(200, gin.H{"message": "中间件链"})
})
```

### 6. 错误处理

```go
// 自定义错误
type APIError struct {
    Code    int    `json:"code"`
    Message string `json:"message"`
}

func Recovery() gin.HandlerFunc {
    return func(c *gin.Context) {
        defer func() {
            if err := recover(); err != nil {
                c.JSON(500, APIError{
                    Code:    500,
                    Message: "服务器内部错误",
                })
                c.Abort()
            }
        }()
        c.Next()
    }
}

// 使用
r.Use(Recovery())
```

### 7. 完整示例

```go
package main

import (
    "github.com/gin-gonic/gin"
    "net/http"
)

type User struct {
    ID    string `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
}

var users = []User{
    {ID: "1", Name: "韩道友", Email: "han@example.com"},
}

func main() {
    r := gin.Default()

    // 获取所有用户
    r.GET("/users", func(c *gin.Context) {
        c.JSON(http.StatusOK, users)
    })

    // 获取单个用户
    r.GET("/users/:id", func(c *gin.Context) {
        id := c.Param("id")
        for _, user := range users {
            if user.ID == id {
                c.JSON(http.StatusOK, user)
                return
            }
        }
        c.JSON(http.StatusNotFound, gin.H{"error": "用户不存在"})
    })

    // 创建用户
    r.POST("/users", func(c *gin.Context) {
        var user User
        if err := c.ShouldBindJSON(&user); err != nil {
            c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
            return
        }
        user.ID = fmt.Sprintf("%d", len(users)+1)
        users = append(users, user)
        c.JSON(http.StatusCreated, user)
    })

    r.Run(":8080")
}
```

---

## 二、Gorm - ORM法宝

Gorm是Go最流行的ORM库，操作数据库像操作切片一样简单。

### 1. 快速开始

```go
package main

import (
    "gorm.io/driver/mysql"
    "gorm.io/gorm"
)

type User struct {
    ID    uint   `gorm:"primaryKey"`
    Name  string `gorm:"size:255"`
    Email string `gorm:"uniqueIndex"`
}

func main() {
    // 连接数据库
    dsn := "user:password@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local"
    db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
    if err != nil {
        panic("连接数据库失败")
    }

    // 自动迁移
    db.AutoMigrate(&User{})
}
```

### 2. 模型定义

```go
type User struct {
    ID        uint           `gorm:"primaryKey"`
    Name      string         `gorm:"size:255;not null"`
    Email     string         `gorm:"uniqueIndex"`
    Age       int            `gorm:"default:0"`
    CreatedAt time.Time
    UpdatedAt time.Time
    DeletedAt gorm.DeletedAt `gorm:"index"`  // 软删除
}

// 表名自定义
func (User) TableName() string {
    return "daoist_users"
}
```

### 3. CRUD操作

```go
// 创建
user := User{Name: "韩道友", Email: "han@example.com"}
result := db.Create(&user)  // 返回的result包含错误信息

// 批量创建
users := []User{
    {Name: "韩道友", Email: "han@example.com"},
    {Name: "李道友", Email: "li@example.com"},
}
db.Create(&users)

// 查询 - 单个
var user User
db.First(&user, 1)  // 查询ID为1的用户
db.First(&user, "name = ?", "韩道友")  // 按条件查询

// 查询 - 多个
var users []User
db.Find(&users)  // 查询所有用户
db.Where("age > ?", 18).Find(&users)  // 条件查询

// 更新
db.Model(&user).Update("name", "新名字")  // 更新单个字段
db.Model(&user).Updates(User{Name: "新名字", Age: 25})  // 更新多个字段

// 批量更新
db.Model(&User{}).Where("active = ?", true).Update("status", "inactive")

// 删除
db.Delete(&user, 1)  // 软删除
db.Unscoped().Delete(&user, 1)  // 硬删除
```

### 4. 查询构建器

```go
// Where条件
db.Where("name = ?", "韩道友").First(&user)
db.Where("name LIKE ?", "%韩%").Find(&users)
db.Where("age >= ? AND age <= ?", 18, 30).Find(&users)

// Or条件
db.Where("name = ?", "韩道友").Or("name = ?", "李道友").Find(&users)

// Not条件
db.Not("name = ?", "韩道友").Find(&users)

// In条件
db.Where("id IN ?", []int{1, 2, 3}).Find(&users)

// 排序
db.Order("age desc").Find(&users)
db.Order("age desc, name asc").Find(&users)

// 限制
db.Limit(10).Find(&users)  // 限制10条
db.Offset(10).Limit(10).Find(&users)  // 跳过10条，取10条

// 计数
var count int64
db.Model(&User{}).Count(&count)

// Pluck - 查询单个字段
var names []string
db.Model(&User{}).Pluck("name", &names)

// Select - 指定字段
db.Select("name", "email").Find(&users)
```

### 5. 关联关系

```go
// 一对多
type User struct {
    ID    uint
    Name  string
    Posts []Post  // 一个用户有多个帖子
}

type Post struct {
    ID     uint
    Title  string
    UserID uint
    User   User  // 帖子属于一个用户
}

// 查询关联
var user User
db.Preload("Posts").First(&user, 1)  // 预加载帖子

// 创建关联
db.Create(&User{
    Name: "韩道友",
    Posts: []Post{
        {Title: "第一篇"},
        {Title: "第二篇"},
    },
})

// 多对多
type Student struct {
    ID        uint
    Name      string
    Courses   []Course `gorm:"many2many:student_courses;"`
}

type Course struct {
    ID       uint
    Name     string
    Students []Student `gorm:"many2many:student_courses;"`
}

// 查询多对多
var student Student
db.Preload("Courses").First(&student, 1)
```

### 6. 事务

```go
// 自动事务
err := db.Transaction(func(tx *gorm.DB) error {
    if err := tx.Create(&User{Name: "韩道友"}).Error; err != nil {
        return err  // 会自动回滚
    }
    if err := tx.Create(&User{Name: "李道友"}).Error; err != nil {
        return err
    }
    return nil  // 提交事务
})

// 手动事务
tx := db.Begin()
defer func() {
    if r := recover(); r != nil {
        tx.Rollback()
    }
}()

if err := tx.Create(&User{Name: "韩道友"}).Error; err != nil {
    tx.Rollback()
    return err
}

tx.Commit()
```

### 7. 钩子函数

```go
type User struct {
    ID   uint
    Name string
}

// 创建前
func (u *User) BeforeCreate(tx *gorm.DB) error {
    fmt.Println("即将创建用户:", u.Name)
    return nil
}

// 创建后
func (u *User) AfterCreate(tx *gorm.DB) error {
    fmt.Println("用户已创建:", u.ID)
    return nil
}

// 更新前/后、删除前/后同理
func (u *User) BeforeUpdate(tx *gorm.DB) error { return nil }
func (u *User) AfterUpdate(tx *gorm.DB) error  { return nil }
func (u *User) BeforeDelete(tx *gorm.DB) error { return nil }
func (u *User) AfterDelete(tx *gorm.DB) error  { return nil }
```

---

## 三、Echo - 轻量级框架

Echo是另一个流行的Web框架，更轻量、性能更高。

```go
package main

import (
    "github.com/labstack/echo/v4"
    "github.com/labstack/echo/v4/middleware"
    "net/http"
)

func main() {
    e := echo.New()

    // 中间件
    e.Use(middleware.Logger())
    e.Use(middleware.Recover())

    // 路由
    e.GET("/", func(c echo.Context) error {
        return c.String(http.StatusOK, "Hello, World!")
    })

    e.GET("/users/:id", func(c echo.Context) error {
        id := c.Param("id")
        return c.JSON(http.StatusOK, map[string]string{"id": id})
    })

    e.POST("/users", func(c echo.Context) error {
        u := new(User)
        if err := c.Bind(u); err != nil {
            return err
        }
        return c.JSON(http.StatusCreated, u)
    })

    // 启动
    e.Logger.Fatal(e.Start(":8080"))
}
```

---

## 四、Viper - 配置管理

Viper是Go最流行的配置管理库，支持多种格式。

```go
package main

import (
    "github.com/spf13/viper"
)

type Config struct {
    Server struct {
        Port int    `mapstructure:"port"`
        Host string `mapstructure:"host"`
    }
    Database struct {
        Host     string `mapstructure:"host"`
        Port     int    `mapstructure:"port"`
        User     string `mapstructure:"user"`
        Password string `mapstructure:"password"`
        DBName   string `mapstructure:"dbname"`
    }
}

func LoadConfig() (*Config, error) {
    viper.SetConfigName("config")  // config.yaml/config.json
    viper.SetConfigType("yaml")
    viper.AddConfigPath(".")
    viper.AddConfigPath("./config")

    // 环境变量
    viper.AutomaticEnv()

    if err := viper.ReadInConfig(); err != nil {
        return nil, err
    }

    var config Config
    if err := viper.Unmarshal(&config); err != nil {
        return nil, err
    }

    return &config, nil
}
```

---

## 五、Logrus - 日志库

```go
package main

import (
    "github.com/sirupsen/logrus"
    "os"
)

func main() {
    log := logrus.New()

    // 设置输出
    log.SetOutput(os.Stdout)

    // 设置格式
    log.SetFormatter(&logrus.JSONFormatter{})
    // log.SetFormatter(&logrus.TextFormatter{})

    // 设置日志级别
    log.SetLevel(logrus.DebugLevel)

    // 使用
    log.WithFields(logrus.Fields{
        "event": "修炼",
        "topic": "筑基",
    }).Info("开始修炼")

    log.Debug("调试信息")
    log.Info("普通信息")
    log.Warn("警告信息")
    log.Error("错误信息")
    log.Fatal("致命错误")
}
```

---

**框架心法口诀：**
Gin路由中间件，Gorm操作数据库
Echo轻量高性能，Viper配置管理
Logrus日志记录，框架组合威力大

道友，框架篇可还满意？💪
