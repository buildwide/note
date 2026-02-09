go env -w GOPROXY=https://goproxy.cn,direct
go mod download

# 含错误处理的 平方根函数
- 这是一个关于 Go 语言中 错误处理（error） 的经典练习。我们需要：
- 定义一个自定义错误类型 ErrNegativeSqrt；
- 为它实现 Error() string 方法，使其满足 error 接口；
- 修改 Sqrt 函数，当输入为负数时返回这个错误；
- 正确实现 Error() 方法以避免死循环。
```go
package main

import (
	"fmt"
)
type ErrNegativeSqrt float64

// 实现 error 接口的 Error 方法
func (e ErrNegativeSqrt) Error() string {
	return fmt.Sprintf("cannot Sqrt negative number: %g", float64(e))
}

func Sqrt(x float64) (float64, error) {
	if x < 0 {
		return 0, ErrNegativeSqrt(x)
	}

	// 使用牛顿法计算平方根
	z := x // 初始猜测值
	for i := 0; i < 10; i++ {
		z -= (z*z - x) / (2 * z)
	}
	return z, nil
}

func main() {
	fmt.Println(Sqrt(2))
	fmt.Println(Sqrt(-2))
}

```
❓ 为什么 fmt.Sprint(e) 会导致死循环？
因为：

    e 的类型是 ErrNegativeSqrt；
    当你调用 fmt.Sprint(e) 时，fmt 包会检查 e 是否实现了 error 接口（确实实现了）；
    为了打印它，fmt 会自动调用 e.Error()；
    而在 Error() 方法内部又调用了 fmt.Sprint(e)；
    这就形成了 无限递归 → 栈溢出 → 程序崩溃。

✅ 解决方法：
将 e 转换为基础类型（如 float64(e)），这样 fmt.Sprint(float64(e)) 就不会触发 Error() 方法，而是直接打印数值。

    所以正确写法是：fmt.Sprintf("...", float64(e))

✅ 总结要点：

    Go 中的错误通过 error 接口表示：type error interface { Error() string }
    自定义错误类型通常基于基本类型（如 float64, string）并实现 Error() 方法；
    在 Error() 方法中不要直接打印自身类型，要转成基础类型避免递归；