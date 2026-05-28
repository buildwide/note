# Go语言进阶心法

## 一、反射 - 看破虚妄

### 1. 基础反射

```go
package main

import (
    "fmt"
    "reflect"
)

// 获取类型信息
func reflectType(x interface{}) {
    t := reflect.TypeOf(x)
    fmt.Printf("类型: %v, 种类: %v\n", t, t.Kind())
    
    // 判断类型
    switch t.Kind() {
    case reflect.Struct:
        fmt.Println("是结构体")
    case reflect.Slice:
        fmt.Println("是切片")
    case reflect.Map:
        fmt.Println("是映射")
    case reflect.Ptr:
        fmt.Println("是指针")
        fmt.Printf("指向: %v\n", t.Elem())
    case reflect.Func:
        fmt.Println("是函数")
    }
}

// 获取值信息
func reflectValue(x interface{}) {
    v := reflect.ValueOf(x)
    fmt.Printf("值: %v\n", v)
    fmt.Printf("可设置: %v\n", v.CanSet())
    fmt.Printf("可寻址: %v\n", v.CanAddr())
    
    // 修改值（需要指针）
    if v.Kind() == reflect.Ptr && !v.IsNil() {
        v = v.Elem()
        if v.CanSet() {
            if v.Kind() == reflect.Int {
                v.SetInt(999)
            }
        }
    }
}

// 遍历结构体字段
func reflectStruct(x interface{}) {
    v := reflect.ValueOf(x)
    t := v.Type()
    
    if t.Kind() != reflect.Struct {
        fmt.Println("不是结构体")
        return
    }
    
    for i := 0; i < v.NumField(); i++ {
        field := t.Field(i)
        value := v.Field(i)
        
        fmt.Printf("字段名: %s, 类型: %v, 值: %v, tag: %s\n",
            field.Name, field.Type, value, field.Tag)
    }
}

// 遍历切片/映射
func reflectSlice(x interface{}) {
    v := reflect.ValueOf(x)
    
    switch v.Kind() {
    case reflect.Slice, reflect.Array:
        for i := 0; i < v.Len(); i++ {
            fmt.Printf("[%d] = %v\n", i, v.Index(i))
        }
    case reflect.Map:
        for _, key := range v.MapKeys() {
            value := v.MapIndex(key)
            fmt.Printf("%v = %v\n", key, value)
        }
    }
}

// 调用方法
func reflectMethod(x interface{}, methodName string, args ...interface{}) {
    v := reflect.ValueOf(x)
    method := v.MethodByName(methodName)
    
    if !method.IsValid() {
        fmt.Printf("方法不存在: %s\n", methodName)
        return
    }
    
    // 准备参数
    in := make([]reflect.Value, len(args))
    for i, arg := range args {
        in[i] = reflect.ValueOf(arg)
    }
    
    // 调用方法
    out := method.Call(in)
    
    // 处理返回值
    for i, v := range out {
        fmt.Printf("返回值[%d]: %v\n", i, v.Interface())
    }
}

// 使用示例
type Cultivator struct {
    Name  string `json:"name" db:"name"`
    Level string `json:"level" db:"level"`
    Power int    `json:"power" db:"power"`
}

func (c Cultivator) Greet() {
    fmt.Printf("%s道友幸会！\n", c.Name)
}

func (c *Cultivator) Upgrade() {
    c.Power += 100
}

func main() {
    c := Cultivator{Name: "韩道友", Level: "金丹", Power: 9999}
    
    reflectType(c)
    reflectValue(&c)
    reflectStruct(c)
    reflectMethod(c, "Greet")
    reflectMethod(&c, "Upgrade")
}
```

### 2. 动态创建

```go
// 创建新值
func createNewValue(t reflect.Type) reflect.Value {
    return reflect.New(t).Elem()
}

// 创建切片
func createSlice(elemType reflect.Type, size int) reflect.Value {
    return reflect.MakeSlice(reflect.SliceOf(elemType), size, size)
}

// 创建映射
func createMap(keyType, valueType reflect.Type) reflect.Value {
    return reflect.MakeMap(reflect.MapOf(keyType, valueType))
}

// 创建结构体
func createStruct(structType reflect.Type) reflect.Value {
    return reflect.New(structType).Elem()
}

// 动态调用函数
func callFunc(fn interface{}, args ...interface{}) []interface{} {
    v := reflect.ValueOf(fn)
    t := v.Type()
    
    // 准备参数
    in := make([]reflect.Value, t.NumIn())
    for i := 0; i < t.NumIn(); i++ {
        in[i] = reflect.ValueOf(args[i])
    }
    
    // 调用
    out := v.Call(in)
    
    // 转换返回值
    result := make([]interface{}, len(out))
    for i, v := range out {
        result[i] = v.Interface()
    }
    
    return result
}
```

### 3. 实用反射工具

```go
// 深度复制
func DeepCopy(src, dst interface{}) error {
    srcVal := reflect.ValueOf(src)
    dstVal := reflect.ValueOf(dst)
    
    if dstVal.Kind() != reflect.Ptr || dstVal.IsNil() {
        return fmt.Errorf("dst必须是非nil指针")
    }
    
    dstVal = dstVal.Elem()
    
    if srcVal.Type() != dstVal.Type() {
        return fmt.Errorf("类型不匹配")
    }
    
    copyRecursive(srcVal, dstVal)
    return nil
}

func copyRecursive(src, dst reflect.Value) {
    switch src.Kind() {
    case reflect.Ptr:
        if src.IsNil() {
            return
        }
        dst.Set(reflect.New(src.Elem().Type()))
        copyRecursive(src.Elem(), dst.Elem())
        
    case reflect.Struct:
        for i := 0; i < src.NumField(); i++ {
            copyRecursive(src.Field(i), dst.Field(i))
        }
        
    case reflect.Slice:
        dst.Set(reflect.MakeSlice(src.Type(), src.Len(), src.Cap()))
        for i := 0; i < src.Len(); i++ {
            copyRecursive(src.Index(i), dst.Index(i))
        }
        
    case reflect.Map:
        dst.Set(reflect.MakeMap(src.Type()))
        for _, key := range src.MapKeys() {
            val := src.MapIndex(key)
            newVal := reflect.New(val.Type()).Elem()
            copyRecursive(val, newVal)
            dst.SetMapIndex(key, newVal)
        }
        
    default:
        dst.Set(src)
    }
}

// 结构体转Map
func StructToMap(obj interface{}) (map[string]interface{}, error) {
    v := reflect.ValueOf(obj)
    if v.Kind() == reflect.Ptr {
        v = v.Elem()
    }
    
    if v.Kind() != reflect.Struct {
        return nil, fmt.Errorf("不是结构体")
    }
    
    result := make(map[string]interface{})
    t := v.Type()
    
    for i := 0; i < v.NumField(); i++ {
        field := t.Field(i)
        value := v.Field(i)
        
        // 跳过非导出字段
        if field.PkgPath != "" {
            continue
        }
        
        // 获取tag中的名称
        name := field.Name
        if tag := field.Tag.Get("json"); tag != "" {
            if tag == "-" {
                continue
            }
            name = tag
        }
        
        result[name] = value.Interface()
    }
    
    return result, nil
}

// Map转结构体
func MapToStruct(m map[string]interface{}, obj interface{}) error {
    v := reflect.ValueOf(obj)
    if v.Kind() != reflect.Ptr || v.IsNil() {
        return fmt.Errorf("obj必须是非nil指针")
    }
    
    v = v.Elem()
    if v.Kind() != reflect.Struct {
        return fmt.Errorf("obj必须指向结构体")
    }
    
    t := v.Type()
    
    for i := 0; i < v.NumField(); i++ {
        field := t.Field(i)
        fieldVal := v.Field(i)
        
        // 跳过非导出字段
        if field.PkgPath != "" {
            continue
        }
        
        // 获取tag中的名称
        name := field.Name
        if tag := field.Tag.Get("json"); tag != "" {
            if tag == "-" {
                continue
            }
            name = tag
        }
        
        // 查找map中的值
        val, ok := m[name]
        if !ok {
            continue
        }
        
        // 设置值
        if fieldVal.CanSet() {
            valVal := reflect.ValueOf(val)
            if valVal.Type().ConvertibleTo(fieldVal.Type()) {
                fieldVal.Set(valVal.Convert(fieldVal.Type()))
            }
        }
    }
    
    return nil
}
```

## 二、Unsafe - 禁忌之术

### 1. 指针转换

```go
package main

import (
    "fmt"
    "unsafe"
)

// int64转float64（不安全但快速）
func Int64ToFloat64(i int64) float64 {
    return *(*float64)(unsafe.Pointer(&i))
}

// float64转int64
func Float64ToInt64(f float64) int64 {
    return *(*int64)(unsafe.Pointer(&f))
}

// []byte转string（零拷贝）
func BytesToString(b []byte) string {
    return *(*string)(unsafe.Pointer(&b))
}

// string转[]byte（零拷贝，只读）
func StringToBytes(s string) []byte {
    return *(*[]byte)(unsafe.Pointer(
        &struct {
            string
            int
        }{s, len(s)},
    ))
}

// 切片转换
func ConvertSlice(src, dst interface{}) {
    srcSlice := (*[2]uintptr)(unsafe.Pointer(&src))
    dstSlice := (*[2]uintptr)(unsafe.Pointer(&dst))
    dstSlice[0] = srcSlice[0]
    dstSlice[1] = srcSlice[1]
}

// 获取结构体偏移量
func GetFieldOffset(structType interface{}, fieldName string) uintptr {
    v := reflect.ValueOf(structType)
    t := v.Type()
    
    if t.Kind() == reflect.Ptr {
        t = t.Elem()
    }
    
    field, ok := t.FieldByName(fieldName)
    if !ok {
        panic("字段不存在")
    }
    
    return field.Offset
}
```

### 2. 内存操作

```go
// 直接内存读写
type Header struct {
    Data uintptr
    Len  int
    Cap  int
}

// 获取切片底层数组指针
func SliceData(slice []int) unsafe.Pointer {
    return unsafe.Pointer(&slice[0])
}

// 创建切片（绕过类型检查）
func MakeSliceFromPointer(ptr unsafe.Pointer, len, cap int) []int {
    s := make([]int, len, cap)
    h := (*Header)(unsafe.Pointer(&s))
    h.Data = uintptr(ptr)
    return s
}

// 字符串拼接（零拷贝）
func FastJoin(strs []string) string {
    totalLen := 0
    for _, s := range strs {
        totalLen += len(s)
    }
    
    buf := make([]byte, totalLen)
    pos := 0
    
    for _, s := range strs {
        copy(buf[pos:], s)
        pos += len(s)
    }
    
    return BytesToString(buf)
}

// 结构体字段直接访问
type Cultivator struct {
    Name  string
    Level string
    Power int
}

func (c *Cultivator) GetPower() int {
    // 直接访问字段，绕过反射
    return *(*int)(unsafe.Pointer(uintptr(unsafe.Pointer(c)) + unsafe.Offsetof(c.Power)))
}

func (c *Cultivator) SetPower(power int) {
    *(*int)(unsafe.Pointer(uintptr(unsafe.Pointer(c)) + unsafe.Offsetof(c.Power))) = power
}
```

### 3. 性能优化

```go
// 快速字符串比较
func FastEqual(a, b string) bool {
    if len(a) != len(b) {
        return false
    }
    
    // 按字比较
    return *(*string)(unsafe.Pointer(&a)) == *(*string)(unsafe.Pointer(&b))
}

// 快速哈希
func FastHash(s string) uint64 {
    // 使用unsafe直接访问字节
    b := *(*[]byte)(unsafe.Pointer(
        &struct {
            string
            int
        }{s, len(s)},
    ))
    
    var h uint64 = 5381
    for _, c := range b {
        h = ((h << 5) + h) + uint64(c)
    }
    
    return h
}

// 对象池（绕过GC）
type ObjectPool struct {
    pool []unsafe.Pointer
    size int
}

func NewObjectPool(size int) *ObjectPool {
    return &ObjectPool{
        pool: make([]unsafe.Pointer, size),
        size: size,
    }
}

func (p *ObjectPool) Get() unsafe.Pointer {
    for i, ptr := range p.pool {
        if ptr != nil {
            p.pool[i] = nil
            return ptr
        }
    }
    return nil
}

func (p *ObjectPool) Put(ptr unsafe.Pointer) bool {
    for i := range p.pool {
        if p.pool[i] == nil {
            p.pool[i] = ptr
            return true
        }
    }
    return false
}
```

### ⚠️ Unsafe警告

```go
// 危险操作示例（仅作演示，切勿使用！）

// 1. 修改只读内存
func Danger1() {
    s := "hello"
    b := StringToBytes(s)
    b[0] = 'H'  // panic: 因为字符串是只读的
}

// 2. 类型不匹配
func Danger2() {
    i := 42
    f := Int64ToFloat64(int64(i))  // 结果是错误的
    fmt.Println(f)  // 不是 42.0
}

// 3. 越界访问
func Danger3() {
    arr := [3]int{1, 2, 3}
    ptr := unsafe.Pointer(&arr[0])
    
    // 访问越界
    val := *(*int)(unsafe.Pointer(uintptr(ptr) + 16))  // 未定义行为
    fmt.Println(val)
}

// 4. GC问题
func Danger4() {
    s := "hello"
    ptr := unsafe.Pointer(&s)
    
    // 如果s被GC回收，ptr就变成了悬空指针
    runtime.GC()
    fmt.Println(*(*string)(ptr))  // 可能崩溃
}
```

## 三、CGO - 跨界融合

### 1. 基础调用

```go
// hello.c
#include <stdio.h>

void hello() {
    printf("Hello from C!\n");
}

int add(int a, int b) {
    return a + b;
}

void print_array(int* arr, int len) {
    for (int i = 0; i < len; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
}
```

```go
// hello.go
package main

/*
#cgo CFLAGS: -O2
#cgo LDFLAGS: -lm
#include "hello.h"
*/
import "C"
import (
    "fmt"
    "unsafe"
)

func main() {
    // 调用C函数
    C.hello()
    
    // 传递参数
    result := C.add(C.int(10), C.int(20))
    fmt.Printf("10 + 20 = %d\n", int(result))
    
    // 传递数组
    arr := []C.int{1, 2, 3, 4, 5}
    C.print_array((*C.int)(unsafe.Pointer(&arr[0])), C.int(len(arr)))
}
```

### 2. 字符串处理

```go
// string.c
#include <string.h>
#include <stdlib.h>

char* reverse_string(char* s) {
    int len = strlen(s);
    char* result = malloc(len + 1);
    
    for (int i = 0; i < len; i++) {
        result[i] = s[len - 1 - i];
    }
    result[len] = '\0';
    
    return result;
}

void free_string(char* s) {
    free(s);
}
```

```go
// string.go
package main

/*
#include <stdlib.h>
#include "string.h"
*/
import "C"
import (
    "fmt"
    "unsafe"
)

func ReverseString(s string) string {
    cStr := C.CString(s)
    defer C.free(unsafe.Pointer(cStr))
    
    result := C.reverse_string(cStr)
    defer C.free_string(result)
    
    return C.GoString(result)
}

func main() {
    s := "修仙之路"
    reversed := ReverseString(s)
    fmt.Printf("%s -> %s\n", s, reversed)
}
```

### 3. 回调函数

```go
// callback.c
#include <stdio.h>

typedef void (*callback_func)(int);

void register_callback(callback_func cb) {
    printf("注册回调函数\n");
    cb(42);
}

void trigger_callbacks(callback_func* cbs, int count) {
    for (int i = 0; i < count; i++) {
        cbs[i](i);
    }
}
```

```go
// callback.go
package main

/*
#include "callback.h"
*/
import "C"
import (
    "fmt"
    "unsafe"
)

//export goCallback
func goCallback(value C.int) {
    fmt.Printf("Go回调被调用: %d\n", int(value))
}

func RegisterCallback() {
    C.register_callback((C.callback_func)(C.goCallback))
}

func main() {
    RegisterCallback()
}
```

### 4. 结构体互操作

```go
// person.h
typedef struct {
    char name[50];
    int age;
    double power;
} Person;

Person* create_person(const char* name, int age, double power);
void free_person(Person* p);
```

```go
// person.go
package main

/*
#include "person.h"
*/
import "C"
import (
    "fmt"
    "unsafe"
)

type Person struct {
    Name  string
    Age   int
    Power float64
}

func NewPerson(name string, age int, power float64) *Person {
    cName := C.CString(name)
    defer C.free(unsafe.Pointer(cName))
    
    cPerson := C.create_person(cName, C.int(age), C.double(power))
    
    return &Person{
        Name:  C.GoString(&cPerson.name[0]),
        Age:   int(cPerson.age),
        Power: float64(cPerson.power),
    }
}

func (p *Person) ToC() *C.Person {
    cName := C.CString(p.Name)
    
    cPerson := (*C.Person)(C.malloc(C.size_t(unsafe.Sizeof(C.Person{}))))
    cPerson.age = C.int(p.Age)
    cPerson.power = C.double(p.Power)
    
    // 复制字符串
    copy((*[50]byte)(unsafe.Pointer(&cPerson.name[0]))[:], []byte(cName))
    
    C.free(unsafe.Pointer(cName))
    return cPerson
}

func main() {
    p := NewPerson("韩道友", 25, 9999.99)
    fmt.Printf("%+v\n", p)
    
    cPerson := p.ToC()
    defer C.free_person(cPerson)
}
```

## 四、泛型 - 万法归宗（Go 1.18+）

### 1. 基础泛型

```go
// 泛型函数
func Max[T comparable](a, b T) T {
    if a > b {
        return a
    }
    return b
}

func Min[T comparable](a, b T) T {
    if a < b {
        return a
    }
    return b
}

// 使用
func main() {
    fmt.Println(Max(10, 20))      // 20
    fmt.Println(Max(3.14, 2.71))  // 3.14
    fmt.Println(Max("a", "b"))    // b
}

// 泛型切片
func Reverse[T any](s []T) []T {
    for i, j := 0, len(s)-1; i < j; i, j = i+1, j-1 {
        s[i], s[j] = s[j], s[i]
    }
    return s
}

func Filter[T any](s []T, pred func(T) bool) []T {
    result := make([]T, 0)
    for _, item := range s {
        if pred(item) {
            result = append(result, item)
        }
    }
    return result
}

func Map[T, U any](s []T, fn func(T) U) []U {
    result := make([]U, len(s))
    for i, item := range s {
        result[i] = fn(item)
    }
    return result
}

func Reduce[T, U any](s []T, init U, fn func(U, T) U) U {
    result := init
    for _, item := range s {
        result = fn(result, item)
    }
    return result
}
```

### 2. 类型约束

```go
// 约束：可排序
type Ordered interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64 |
        ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 |
        ~float32 | ~float64 |
        ~string
}

func Sort[T Ordered](s []T) {
    for i := 0; i < len(s); i++ {
        for j := i + 1; j < len(s); j++ {
            if s[i] > s[j] {
                s[i], s[j] = s[j], s[i]
            }
        }
    }
}

// 约束：数值类型
type Number interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64 |
        ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 |
        ~float32 | ~float64
}

func Sum[T Number](nums []T) T {
    var total T
    for _, n := range nums {
        total += n
    }
    return total
}

func Average[T Number](nums []T) float64 {
    if len(nums) == 0 {
        return 0
    }
    sum := Sum(nums)
    return float64(sum) / float64(len(nums))
}

// 约束：自定义接口
type Stringer interface {
    String() string
}

type Shaper interface {
    Area() float64
}

func PrintStringers[T Stringer](items []T) {
    for _, item := range items {
        fmt.Println(item.String())
    }
}

func TotalArea[T Shaper](shapes []T) float64 {
    total := 0.0
    for _, shape := range shapes {
        total += shape.Area()
    }
    return total
}
```

### 3. 泛型结构体

```go
// 泛型栈
type Stack[T any] struct {
    items []T
}

func NewStack[T any]() *Stack[T] {
    return &Stack[T]{
        items: make([]T, 0),
    }
}

func (s *Stack[T]) Push(item T) {
    s.items = append(s.items, item)
}

func (s *Stack[T]) Pop() (T, bool) {
    if len(s.items) == 0 {
        var zero T
        return zero, false
    }
    
    index := len(s.items) - 1
    item := s.items[index]
    s.items = s.items[:index]
    return item, true
}

func (s *Stack[T]) Peek() (T, bool) {
    if len(s.items) == 0 {
        var zero T
        return zero, false
    }
    return s.items[len(s.items)-1], true
}

func (s *Stack[T]) IsEmpty() bool {
    return len(s.items) == 0
}

func (s *Stack[T]) Size() int {
    return len(s.items)
}

// 泛型缓存
type Cache[K comparable, V any] struct {
    data map[K]V
}

func NewCache[K comparable, V any]() *Cache[K, V] {
    return &Cache[K, V]{
        data: make(map[K]V),
    }
}

func (c *Cache[K, V]) Set(key K, value V) {
    c.data[key] = value
}

func (c *Cache[K, V]) Get(key K) (V, bool) {
    value, ok := c.data[key]
    return value, ok
}

func (c *Cache[K, V]) Delete(key K) {
    delete(c.data, key)
}

func (c *Cache[K, V]) Has(key K) bool {
    _, ok := c.data[key]
    return ok
}

func (c *Cache[K, V]) Clear() {
    c.data = make(map[K]V)
}
```

### 4. 泛型方法

```go
// 泛型接口
type Container[T any] interface {
    Add(item T)
    Remove(item T) bool
    Contains(item T) bool
}

// 泛型实现
type List[T any] struct {
    items []T
}

func (l *List[T]) Add(item T) {
    l.items = append(l.items, item)
}

func (l *List[T]) Remove(item T) bool {
    for i, v := range l.items {
        if v == item {
            l.items = append(l.items[:i], l.items[i+1:]...)
            return true
        }
    }
    return false
}

func (l *List[T]) Contains(item T) bool {
    for _, v := range l.items {
        if v == item {
            return true
        }
    }
    return false
}

// 泛型接收器
type Processor[T any] struct {
    data []T
}

func (p *Processor[T]) Process(fn func(T) T) {
    for i := range p.data {
        p.data[i] = fn(p.data[i])
    }
}

func (p *Processor[T]) Filter(pred func(T) bool) []T {
    result := make([]T, 0)
    for _, item := range p.data {
        if pred(item) {
            result = append(result, item)
        }
    }
    return result
}

func (p *Processor[T]) Map(fn func(T) any) []any {
    result := make([]any, len(p.data))
    for i, item := range p.data {
        result[i] = fn(item)
    }
    return result
}
```

### 5. 实战案例

```go
// 泛型二叉树
type TreeNode[T any] struct {
    Value T
    Left  *TreeNode[T]
    Right *TreeNode[T]
}

type Tree[T any] struct {
    root *TreeNode[T]
    less func(T, T) bool
}

func NewTree[T any](less func(T, T) bool) *Tree[T] {
    return &Tree[T]{
        less: less,
    }
}

func (t *Tree[T]) Insert(value T) {
    t.root = t.insert(t.root, value)
}

func (t *Tree[T]) insert(node *TreeNode[T], value T) *TreeNode[T] {
    if node == nil {
        return &TreeNode[T]{Value: value}
    }
    
    if t.less(value, node.Value) {
        node.Left = t.insert(node.Left, value)
    } else {
        node.Right = t.insert(node.Right, value)
    }
    
    return node
}

func (t *Tree[T]) Inorder(fn func(T)) {
    t.inorder(t.root, fn)
}

func (t *Tree[T]) inorder(node *TreeNode[T], fn func(T)) {
    if node == nil {
        return
    }
    
    t.inorder(node.Left, fn)
    fn(node.Value)
    t.inorder(node.Right, fn)
}

// 使用
func main() {
    tree := NewTree(func(a, b int) bool { return a < b })
    
    tree.Insert(5)
    tree.Insert(3)
    tree.Insert(7)
    tree.Insert(1)
    tree.Insert(9)
    
    tree.Inorder(func(val int) {
        fmt.Println(val)
    })
}
```

---

**进阶要诀：**
- 反射强大但性能差，能用泛型就不用反射
- unsafe是双刃剑，性能提升伴随风险，谨慎使用
- CGO是跨界术，能调用C库但要注意内存管理
- 泛型让代码更通用，但不要过度泛化
- 性能优化先测量，别凭感觉瞎改

道友，进阶篇记住了吗？这些是禁忌之术，用好了如虎添翼，用不好走火入魔！🌙
