# 用golang写gee(day6)

利用`http`包的`http.FileServer`，结合动态路由来实现`Static`方法，随意访问任意路径下的文件。
<!--more-->
```go
// create static handler  
func (group *RouterGroup) createStaticHandler(relativePath string, fs http.FileSystem) HandlerFunc {  
	absolutePath := path.Join(group.prefix, relativePath)  
	fileServer := http.StripPrefix(absolutePath, http.FileServer(fs))  
	return func(c *Context) {  
		file := c.Param("filepath")  
		// Check if file exists and/or if we have permission to access it  
		if _, err := fs.Open(file); err != nil {  
			c.Status(http.StatusNotFound)  
			return  
		}  
  
		fileServer.ServeHTTP(c.Writer, c.Req)  
	}  
}  
  
// serve static files  
func (group *RouterGroup) Static(relativePath string, root string) {  
	handler := group.createStaticHandler(relativePath, http.Dir(root))  
	urlPattern := path.Join(relativePath, "/*filepath")  
	// Register GET handlers  
	group.GET(urlPattern, handler)  
}
```

代码非常浅显易懂。

模板的知识不太了解，暂时不做。

---

> 作者: [hongwei](https://github.com/hongwei7)  
> URL: https://hongwei7.online/%E7%94%A8golang%E5%86%99geeday6/  

