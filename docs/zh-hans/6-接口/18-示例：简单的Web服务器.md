<center><h1>示例：简单的Web服务器</h1></center>

---

使用 Go 语言的库非常容易实现一个 Web 服务器，用来响应像 fetch 那样的客户端请求。本节将展示一个迷你服务器，返回访问服务器的 URL 的路径部分。例如，如果请求的 URL 是 `http://localhost:8000/hello` ，响应将是 `URL.Path = "/hello"` 。

```go
// server1.go 这是一个迷你回声服务器
package main
import (
    "fmt"
    "log"
    "net/http"
)
func main() {
    http.HandleFunc("/", handler) //回声请求调用处理程序
    log.Fatal(http.ListenAndServe("localhost:8000", nil))
}
//处理程序回显请求 URL r 的路径部分
func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "URL.Path = %q\n", r.URL.Path)
}
```

这个程序只有寥寥几行代码，因为库函数做了大部分工作。main 函数将一个处理函数和以“/”开头的 URL 链接在一起，代表所有的 URL 使用这个函数处理，然后启动服务器监听进入 8000 端口处的请求。

一个请求由一个 http.Request 类型的结构体表示，它包含很多关联的域，其中一个是所请求的 URL。当一个请求到达时，它被转交给处理函数，并从请求的 URL 中提取路径部分 (/hello)，使用 fmt.Printf 格式化，然后作为响应发送回去。

让我们在后台启动服务器。在 Mac OS X 或者 Linux 上，在命令行后添加一个 & 符号；在微软 Windows 上，不需要 & 符号，而需要单独开启一个独立的命令行窗口。

```
$ go run src/gopl.io/chl/serverl/main.go &
```

可以从命令行发起客户请求：

```
$ go build gopl.io/ch1/fetch
$ ./fetch http://localhost:8000
URL.Path = T
$ ./fetch http://localhost:8000/help
URL.Path = "/help"
```

为服务器添加功能很容易。一个有用的扩展是一个特定的 URL，它返回某种排序的状态。例如，这个版本的程序完成和回声服务器一样的事情，但同时返回请求的数量；URL /count 请求返回到现在为止的个数，去掉 /count 请求本身：

```go
//server2.go 这是一个迷你的回声和计数器服务器
package main
import (
    "fmt"
    "log"
    "net/http"
    "sync"
)
var mu sync.Mutex
var count int
func main() {
    http.HandleFunc("/", handler)
    http.HandleFunc("/count", counter)
    log.Fatal(http.ListenAndServe("localhost:8000", nil))
}
//处理程序回显请求的 URL 的路径部分
func handler(w http.ResponseWriter, r *http.Request) {
    mu.Lock()
    count++
    mu.Unlock()
    fmt.Fprintf(w, "URL.Path = %q\n", r.URL.Path)
}
// counter 回显目前为止调用的次数
func counter(w http.ResponseWriter, r *http.Request) {
    mu.Lock()
    fmt.Fprintf(w, "Count %d\n", count)
    mu.Unlock()
}
```

这个服务器有两个处理函数，通过请求的 URL 来决定哪一个被调用：请求 /count 调用 counter，其他的调用 handler。以“/”结尾的处理模式匹配所有含有这个前缀的 URL。在后台，对于每个传入的请求，服务器在不同的 goroutine 中运行该处理函数，这样它可以同时处理多个请求。

然而，如果两个并发的请求试图同时更新计数值 count，它可能会不一致地增加，程序会产生一个严重的竞态 bug。为避免该问题，必须确保最多只有一个 goroutine 在同一时间访问变量，这正是 mu.Lock() 和 mu.Unlock() 语句的作用。

作为一个更完整的例子，处理函数可以报告它接收到的消息头和表单数据，这样可以方便服务器审查和调试请求：

```go
//处理程序回显 HTTP 请求
func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "%s %s %s\n", r.Method, r.URL, r.Proto)
    for k, v := range r.Header {
        fmt.Fprintf(w, "Header[%q] = %q\n", k, v)
    }
    fmt.Fprintf(w, "Host = %q\n", r.Host)
    fmt.Fprintf(w, "RemoteAddr = %q\n", r.RemoteAddr)
    if err := r.ParseForm(); err != nil {
        log.Print(err)
    }
    for k, v := range r.Form {
        fmt.Fprintf(w, "Form[%q] = %q\n", k, v)
    }
}
```

这里使用 http.Request 结构体的成员来产生类似下面的输出：

```
GET /?q=query HTTP/1.1
Header["Accept-Encoding"] = ["gzip, deflate, sdch"]
Header["Accept-Language"] = ["en-US, en;q=0.8"]
Header["Connection"] = ["keep-alive"]
Header["Accept"] = ["text/html, application/xhtml+xml, application/xml; ..."]
Header["User-Agent"] = ["Mozilla/5.0 (Macintosh; Intel Mac OS X 10_7_5)..."]
Host = "localhost:8000"
RemoteAddr = "127.0.0.1:59911"
Form["q"] = ["query"]
```

注意这里是如何在 if 语句中嵌套调用 ParseForm 的。Go 允许一个简单的语句（如一个局部变量声明）跟在 if 条件的前面，这在错误处理的时候特别有用。也可以这样写：

```go
err := r.ParseForm()
if err != nil {
    log.Print(err)
}
```

这些程序中，我们看到了作为输出流的三种非常不同的类型。fetch 程序复制 HTTP 响应到文件 os.Stdout，像 lissajous 一样；fetchall 程序通过将响应复制到 ioutil.Discard 中进行丢弃（在统计其长度时）；Web 服务器使 fmt.Fprintf 通过写入 http.Responsewriter 来让浏览器显示。

尽管三种类型细节不同，但都满足一个通用的接口 (interface)，该接口允许它们按需使用任何一种输出流。该接口称为 io.Writer。

我们来看一下整合 Web 服务器和 lissajous 函数是一件多么容易的事情，这样 GIF 动画将不再输出到标准输出而是 HTTP 客户端。简单添加这些行到 Web 服务器：

```go
handler := func(w http.ResponseWriter, r *http.Request) {
    lissajous(w)
}
http.HandleFunc("/", handler)
```

或者也可以：

```
http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    lissajous(w)
})
```

上面 HandleFunc 函数中立即调用的第二个参数是函数字面量，这是一个在该场景中使用它时才定义的匿名函数。

一旦完成这个改变，就可以通过浏览器访问 `http://localhost:8000` 。每次加载页面，将看到一个类似下图的动画。

<div align=center> 
    <img src="img/6-接口/18-示例：简单的Web服务器/浏览器中的动态利萨茹图形.png"/> 
    <p>图：浏览器中的动态利萨茹图形</p>
</div>
