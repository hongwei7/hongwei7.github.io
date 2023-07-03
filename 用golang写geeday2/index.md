# 用golang写gee(day2)

项目地址：[https://geektutu.com/post/gee-day2.html](https://geektutu.com/post/gee-day2.html)

### 目标
今天的目的是，进一步把`engine`抽象成`router`，且可以使用`JSON`等高度封装的函数，如：

```go
func handler1(c *gee.Context) {
	m := map[string]interface{}{
		"name": "hongwei7",
		"age":  25,
	}
	c.JSON(http.StatusOK, m)
}
```
### 原理
这样的调用无疑是更加方便，更加适合网络开发的。实现的原理，实际上就是将`ResponseWriter`和`Request`打包成`gee.Context`，再实现相应的函数。（不过要注意这里的`Context`和go里面的`Context`不是同一个概念）

```go
type Context struct {
	Writer     http.ResponseWriter
	Req        *http.Request
	Path       string
	Method     string
	StatusCode int
}

func (c *Context) JSON(code int, obj interface{}) {
	c.SetHeader("Content-Type", "application/json")
	c.Status(code)
	encoder := json.NewEncoder(c.Writer)
	if err := encoder.Encode(obj); err != nil {
		http.Error(c.Writer, err.Error(), 500)
	}
}
```

这样的简单封装，省略了各种状态码和返回类型`header`的填写。最后可以这样使用我们的`gee`了：

```go
func helloHandler(c *gee.Context) {
	c.String(http.StatusOK, "hello world")
}

func handler1(c *gee.Context) {
	m := map[string]interface{}{
		"name": "hongwei7",
		"age":  25,
	}
	c.JSON(http.StatusOK, m)
}

func main() {
	gee := gee.New()
	gee.GET("/hello", helloHandler)
	gee.GET("/", handler1)
	gee.Run()
}
```
这就是day2的内容。

---

> 作者: [hongwei](https://github.com/hongwei7)  
> URL: https://hongwei7.online/%E7%94%A8golang%E5%86%99geeday2/  

