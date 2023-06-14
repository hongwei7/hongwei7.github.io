# 用golang写gee(day1)


项目地址：[https://geektutu.com/post/gee-day1.html](https://geektutu.com/post/gee-day1.html)
### 实现

第一天主要介绍`net/http`模块的用法，实现了里面的`Handler` interface，从而接管了整个`http`库的行为。

```go
// net/http主要进入函数
func ListenAndServe(addr string, handler Handler) error {
	server := &Server{Addr: addr, Handler: handler}
	return server.ListenAndServe()
}

// Handler接口如下
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}
```

所以自己完全可以封装一个`Handler`。另外简单实现了一个`string map router`，记录了所有的处理函数，并实现了添加和调用这些处理函数的接口。核心封装函数如下：

```go
type HandlerFunc func(http.ResponseWriter, *http.Request)

type Engine struct {
	router map[string]HandlerFunc
}

func New() *Engine {
	return &Engine{make(map[string]HandlerFunc)}
}

func (engine *Engine) addRouter(method string, pattern string, handler HandlerFunc) {
	key := method + "-" + pattern
	engine.router[key] = handler
}

func (engine *Engine) GET(pattern string, handler HandlerFunc) {
	engine.addRouter("GET", pattern, handler)
}

func (engine *Engine) POST(pattern string, handler HandlerFunc) {
	engine.addRouter("POST", pattern, handler)
}

func (engine *Engine) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	key := req.Method + "-" + req.URL.Path
	if handler, ok := engine.router[key]; ok {
		handler(w, req)
	} else {
		fmt.Fprintf(w, "404 NOT FOUND: %q", req.URL.Path)
	}
}

func (engine *Engine) Run() {
	http.ListenAndServe(":9999", engine)
}
```

### 总结

第一天的内容还是较为简单的，主要是简单封装了一下`net/http`库。还有就是感觉用`go`写出来的代码都长得差不多，像`func New()`这种写法都是固定的。

---

> 作者: [hongwei](https://github.com/hongwei7)  
> URL: https://hongwei7.online/%E7%94%A8golang%E5%86%99geeday1/  

