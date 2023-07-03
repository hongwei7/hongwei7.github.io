# 用golang写gee(day5)

项目地址：[https://geektutu.com/post/gee-day5.html](https://geektutu.com/post/gee-day5.html)
<!--more-->
今天的目的是在`Group`中嵌入中间件。中间件的使用方式为：
```go
func (r *RouterGroup) Use(middlewares ...HandlerFunc)
```
对于两个中间件`A,B`，嵌套后的调用顺序为：`part1 -> part3 -> Handler -> part 4 -> part2`
```go
func A(c *Context) {  
    part1  
    c.Next()  
    part2  
}  
func B(c *Context) {  
    part3  
    c.Next()  
    part4  
}
```
实际上，访问资源时，`middleware`从`Group`传递到`Context`，而`Context`被`handle`时，加入注册于该`Group`的`handler`（使用`method+pattern`注册的），再调用`c.Next()`来开始运行这一系列函数表（`middlewares+handler`）。
```go

func (r *router) handle(c *Context) {
	n, params := r.getRoute(c.Method, c.Path)
	if n != nil {
			c.Params = params
			key := c.Method + "-" + n.pattern
			c.handlers = append(c.handlers, r.handlers[key])
		} else {
			c.handlers = append(c.handlers, func(c *Context) {
			c.String(http.StatusNotFound, "404 NOT FOUND")
		})
	}
	c.Next()
}

func (c *Context) Next() {
	c.index++
	s := len(c.handlers)
	for ; c.index < s; c.index++ {
		c.handlers[c.index](c)
	}
}
```

在运行某个函数时，随时可以使用`c.Next()`来跳出函数，执行`index`指向的下个函数。（这里能产生很多种调用行为顺序，比如可以一个主函数+多个子函数）




---

> 作者: [hongwei](https://github.com/hongwei7)  
> URL: https://hongwei7.online/%E7%94%A8golang%E5%86%99geeday5/  

