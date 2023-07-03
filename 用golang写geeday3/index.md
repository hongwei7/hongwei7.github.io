# 用golang写gee(day3)



项目地址：[https://geektutu.com/post/gee-day3.html](https://geektutu.com/post/gee-day3.html)

### 目标
day3的任务是，实现`http`的动态路由，即访问路径带有`:name`的资源时，可以顺利匹配；访问路径带有`*`的资源时，直接匹配后面所有资源。<!--more-->例如`/p/go/doc`匹配到`/p/:lang/doc`，解析结果为：`{lang: "go"}`，`/static/css/geektutu.css`匹配到`/static/*filepath`，解析结果为`{filepath: "css/geektutu.css"}`。
### 实现
主要是实现一个简单的前缀树来完成。
```go
type node struct {
	pattern string // 待匹配路由 /p/:lang
	part string // :lang
	children []*node
	isWild bool //是否精确匹配,part 含有:或*
}
```
这里`pattern`是该资源的实际路径，而`part`是将该资源路径切开后的切片，如`info`和 `：name`。实现简单的`insert`和`search`操作后，嵌入到`router`里面。
```go
func (r *router) addRoute(method string, pattern string, handler HandlerFunc) {
	parts := parsePattern(pattern)
	_, ok := r.roots[method]
	if !ok {
		r.roots[method] = &node{}
	}
	
	r.roots[method].insert(pattern, parts, 0) //插入到trieTree
	key := method + "-" + pattern
	r.handlers[key] = handler
}

func (r *router) getRoute(method string, path string) (*node, map[string]string) {
	searchParts := parsePattern(path)
	params := make(map[string]string)
	root, ok := r.roots[method]
	if !ok {
		return nil, nil
	}
	
	n := root.search(searchParts, 0) //在trieTree中搜索
	if n != nil {
		parts := parsePattern(n.pattern)
		for index, part := range parts {
			if part[0] == ':' {
				params[part[1:]] = searchParts[index]
			}

			if part[0] == '*' && len(part) > 1 {
				params[part[1:]] = strings.Join(searchParts[index:], "/")
				break
			}
		}
		return n, params
	}
	return nil, nil
}

func (r *router) handle(c *Context) {
	n, params := r.getRoute(c.Method, c.Path)
	if n != nil {
		c.Params = params //更新动态路由参数
		key := c.Method + "-" + n.pattern
		r.handlers[key](c)
	} else {
		c.String(http.StatusNotFound, "404 NOT FOUND")
	}
}
```

只有实际可以访问的节点的`pattern`设置为实际访问路径，**其余父节点为空**，注意这个判断即可。
```go
func (n *node) search(parts []string, height int) *node {  
	if len(parts) == height || strings.HasPrefix(n.part, "*") {  
		if n.pattern == "" {  
			return nil  
		}  
		return n  
	}
```

### 单元测试
```go
func newTestRouter() *router {
	r := newRouter()
	r.addRoute("GET", "/", nil)
	r.addRoute("GET", "/hello/:name", nil)
	r.addRoute("GET", "/hello/b/c", nil)
	r.addRoute("GET", "/hi/:name", nil)
	r.addRoute("GET", "/assets/*filepath", nil)
	return 
}

func TestParsePattern(t *testing.T) {
	ok := reflect.DeepEqual(parsePattern("/p/:name"), []string{"p", ":name"})
	ok = ok && reflect.DeepEqual(parsePattern("/p/*"), []string{"p", "*"})
	ok = ok && reflect.DeepEqual(parsePattern("/p/*name/*"), []string{"p", "*name"})
	if !ok {
		t.Fatal("test parsePattern failed")
	}
}

func TestGetRoute(t *testing.T) {
	r := newTestRouter()
	n, ps := r.getRoute("GET", "/hello/geektutu")
	if n == nil {
		t.Fatal("nil shouldn't be returned")
	}
	if n.pattern != "/hello/:name" {
		t.Fatal("should match /hello/:name")
	}
	if ps["name"] != "geektutu" {
		t.Fatal("name should be equal to 'geektutu'")
	}
	fmt.Printf("matched path: %s, params['name']: %s\n", n.pattern, ps["name"])
}
```

---

> 作者: [hongwei](https://github.com/hongwei7)  
> URL: https://hongwei7.online/%E7%94%A8golang%E5%86%99geeday3/  

