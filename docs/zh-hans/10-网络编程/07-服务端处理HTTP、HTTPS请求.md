<center><h1>服务端处理HTTP、HTTPS请求</h1></center>

---

### 处理 HTTP 请求

使用 net/http 包提供的 http.ListenAndServe() 方法，可以在指定的地址进行监听，开启一个 HTTP，服务端该方法的原型如下：

```go
func ListenAndServe(addr string, handler Handler) error
```

该方法用于在指定的 TCP 网络地址 addr 进行监听，然后调用服务端处理程序来处理传入的连接请求。

该方法有两个参数：第一个参数 addr 即监听地址；第二个参数表示服务端处理程序，通常为空，这意味着服务端调用 http.DefaultServeMux 进行处理，而服务端编写的业务逻辑处理程序 http.Handle() 或 http.HandleFunc() 默认注入 http.DefaultServeMux 中，具体代码如下：

```go
http.Handle("/foo", fooHandler)
http.HandleFunc("/bar", func(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, %q", html.EscapeString(r.URL.Path))
})
log.Fatal(http.ListenAndServe(":8080", nil))
```

如果想更多地控制服务端的行为，可以自定义 http.Server，代码如下：

```go
s := &http.Server{
    Addr: ":8080",
    Handler: myHandler,
    ReadTimeout: 10 * time.Second,
    WriteTimeout: 10 * time.Second,
    MaxHeaderBytes: 1 << 20,
}
log.Fatal(s.ListenAndServe())
```

### 处理 HTTPS 请求

net/http 包还提供 http.ListenAndServeTLS() 方法，用于处理 HTTPS 连接请求：

```go
func ListenAndServeTLS(addr string, certFile string, keyFile string, handler Handler)
    error
```

ListenAndServeTLS() 和 ListenAndServe() 的行为一致，区别在于只处理 HTTPS 请求。

此外，服务器上必须存在包含证书和与之匹配的私钥的相关文件，比如 certFile 对应 SSL 证书文件存放路径，keyFile 对应证书私钥文件路径。如果证书是由证书颁发机构签署的，certFile 参数指定的路径必须是存放在服务器上的经由 CA 认证过的 SSL 证书。

开启 SSL 监听服务也很简单，如下列代码所示：

```go
http.Handle("/foo", fooHandler)
http.HandleFunc("/bar", func(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, %q", html.EscapeString(r.URL.Path))
})
log.Fatal(http.ListenAndServeTLS(":10443", "cert.pem", "key.pem", nil))
```

或者是：

```go
ss := &http.Server{
    Addr: ":10443",
    Handler: myHandler,
    ReadTimeout: 10 * time.Second,
    WriteTimeout: 10 * time.Second,
    MaxHeaderBytes: 1 << 20,
}
log.Fatal(ss.ListenAndServeTLS("cert.pem", "key.pem"))
```
