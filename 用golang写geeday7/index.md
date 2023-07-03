# 用golang写gee(day7)

在 Go 中 panic 会导致程序被中止，但是在退出前，会先处理完当前协程上已经 defer 的任务，执行完成后再退出。效果类似于 java 语言的 `try...catch`。defer 的任务执行完成之后，panic 还会继续被抛出，导致程序非正常结束。

Go 语言还提供了 recover 函数，可以避免因为 panic 发生而导致整个程序终止，recover 函数只在 defer 中生效。
<!--more-->

```go
// hello.go  
func test_recover() {  
	defer func() {  
		fmt.Println("defer func")  
		if err := recover(); err != nil {  
			fmt.Println("recover success")  
		}  
	}()  
  
	arr := []int{1, 2, 3}  
	fmt.Println(arr[4])  
	fmt.Println("after panic")  
}  
  
func main() {  
	test_recover()  
	fmt.Println("after recover")  
}
```

我们要给gee做一个内置的`recovery`机制，使得用户调用的函数出现了错误，也能出现`Internal Server Error`，而不是直接退出进程。

实际上是实现一个`recovery`中间件：
```go
package gee  
  
import (  
	"fmt"  
	"log"  
	"net/http"  
	"runtime"  
	"strings"  
)  
  
// print stack trace for debug  
func trace(message string) string {  
	var pcs [32]uintptr  
	n := runtime.Callers(3, pcs[:]) // skip first 3 caller  
  
	var str strings.Builder  
	str.WriteString(message + "\nTraceback:")  
	for _, pc := range pcs[:n] {  
		fn := runtime.FuncForPC(pc)  
		file, line := fn.FileLine(pc)  
		str.WriteString(fmt.Sprintf("\n\t%s:%d", file, line))  
	}  
	return str.String()  
}  
  
func Recovery() HandlerFunc {  
	return func(c *Context) {  
		defer func() {  
			if err := recover(); err != nil {  
				message := fmt.Sprintf("%s", err)  
				log.Printf("%s\n\n", trace(message))  
				c.Fail(http.StatusInternalServerError, "Internal Server Error")  
			}  
		}()  
  
		c.Next()  
	}  
}
```

gee默认使用`Logger`和`Recovery`中间件，这样就实现了恢复功能。

---

> 作者: [hongwei](https://github.com/hongwei7)  
> URL: https://hongwei7.online/%E7%94%A8golang%E5%86%99geeday7/  

