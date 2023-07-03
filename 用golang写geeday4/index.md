# 用golang写gee(day4)



项目地址：[https://geektutu.com/post/gee-day4.html](https://geektutu.com/post/gee-day4.html)

### 目标
这次要完成http的分组功能，简而言之，就是对于一些具有共同前缀的资源，我们可以通过哦`Group（）`来一起注册，且可以嵌入中间件，使得这些资源都有相同的特性。最终使用效果如下：
```go
func main() {
	r := gee.New()
	r.GET("/index", func(c *gee.Context) {
		c.HTML(http.StatusOK, "<h1>Index Page</h1>")
	})
	v1 := r.Group("/v1")
	{
		v1.GET("/", func(c *gee.Context) {
			c.HTML(http.StatusOK, "<h1>Hello Gee</h1>")
		})

		v1.GET("/hello", func(c *gee.Context) {
			// expect /hello?name=geektutu
			c.String(http.StatusOK, "hello %s, you're at %s\n", c.Query("name"), c.Path)
		})
	}
	v2 := r.Group("/v2")
	{
		v2.GET("/hello/:name", func(c *gee.Context) {
			// expect /hello/geektutu
			c.String(http.StatusOK, "hello %s, you're at %s\n", c.Param("name"), c.Path)
		})
		v2.POST("/login", func(c *gee.Context) {
			c.JSON(http.StatusOK, gee.H{
				"username": c.PostForm("username"),
				"password": c.PostForm("password"),
			})
		})

	}

	r.Run(":9999")
}
```
可以看得出来，这样实现会比较简洁漂亮。也可以使用嵌套和中间件。
### 实现
定义`RouterGroup`类型，`Engine`本身就是一个`RouterGroup`，而所有的`RouterGroup`都指向一个`Engine`。
```go
RouterGroup struct {  
	prefix      string  
	middlewares []HandlerFunc // support middleware  
	parent      *RouterGroup  // support nesting  
	engine      *Engine       // all groups share a Engine instance  
}

Engine struct {  
	*RouterGroup  
	router *router  
	groups []*RouterGroup // store all groups  
}
```
留意`Engine`在初始化时，`Groups`只包含它自身的`RouterGroup`。将原来为`Engine`实现的`GET`等方法，改为为`RouterGroup`的，此时`Engine`也可以隐性调用这些成员函数。这样就可以偷龙转凤一般，实现这种效果：
```go
v1 := r.Group("/v1")
{
	v1.GET("/", func(c *gee.Context) {
		c.HTML(http.StatusOK, "<h1>Hello Gee</h1>")
	})

	v1.GET("/hello", func(c *gee.Context) {
		// expect /hello?name=geektutu
		c.String(http.StatusOK, "hello %s, you're at %s\n", c.Query("name"), c.Path)
	})
}
```

说白了，现在就是加了个前缀功能而已。`Grouping`真正的威力要在中间件中体现出来吧？

---

> 作者: [hongwei](https://github.com/hongwei7)  
> URL: https://hongwei7.online/%E7%94%A8golang%E5%86%99geeday4/  

