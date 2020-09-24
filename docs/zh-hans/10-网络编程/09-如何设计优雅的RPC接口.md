<center><h1>如何设计优雅的RPC接口</h1></center>

---

如何设计优雅的RPC接口

Go语言的 net/rpc 很灵活，它在数据传输前后实现了编码解码器的接口定义。这意味着，开发者可以自定义数据的传输方式以及 RPC 服务端和客户端之间的交互行为。

RPC 提供的编码解码器接口如下：
```go
type ClientCodec interface {
    WriteRequest(*Request, interface{}) error
    ReadResponseHeader(*Response) error
    ReadResponseBody(interface{}) error
    Close() error
}
type ServerCodec interface {
    ReadRequestHeader(*Request) error
    ReadRequestBody(interface{}) error
    WriteResponse(*Response, interface{}) error
    Close() error
}
```
接口 ClientCodec 定义了 RPC 客户端如何在一个 RPC 会话中发送请求和读取响应。客户端程序通过 WriteRequest() 方法将一个请求写入到 RPC 连接中，并通过 ReadResponseHeader() 和 ReadResponseBody() 读取服务端的响应信息。当整个过程执行完毕后，再通过 Close() 方法来关闭该连接。

接口 ServerCodec 定义了 RPC 服务端如何在一个 RPC 会话中接收请求并发送响应。服务端程序通过 ReadRequestHeader() 和 ReadRequestBody() 方法从一个 RPC 连接中读取请求信息。

然后再通过 WriteResponse() 方法向该连接中的 RPC 客户端发送响应。当完成该过程后，通过 Close() 方法来关闭连接。

通过实现上述接口，我们可以自定义数据传输前后的编码解码方式，而不仅仅局限于 Gob。

同样，可以自定义 RPC 服务端和客户端的交互行为。实际上，Go 标准库提供的 net/rpc/json 包，就是一套实现了 rpc.ClientCodec 和 rpc.ServerCodec 接口的 JSON-RPC 模块。

