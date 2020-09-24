<center><h1>示例：建立TCP链接</h1></center>

---

下面我们建立 TCP 链接来实现初步的 HTTP 协议，通过向网络主机发送 HTTP Head 请求，读取网络主机返回的信息，具体代码如下所示。

```go
package main
import (
    "net"
    "os"
    "bytes"
    "fmt"
)
func main() {
    if len(os.Args) != 2 {
        fmt.Fprintf(os.Stderr, "Usage: %s host:port", os.Args[0])
        os.Exit(1)
    }
    service := os.Args[1]
    conn, err := net.Dial("tcp", service)
    checkError(err)
    _, err = conn.Write([]byte("HEAD / HTTP/1.0\r\n\r\n"))
    checkError(err)
    result, err := readFully(conn)
    checkError(err)
    fmt.Println(string(result))
    os.Exit(0)
}
func checkError(err error) {
    if err != nil {
        fmt.Fprintf(os.Stderr, "Fatal error: %s", err.Error())
        os.Exit(1)
    }
}
func readFully(conn net.Conn) ([]byte, error) {
    defer conn.Close()
    result := bytes.NewBuffer(nil)
    var buf [512]byte
    for {
        n, err := conn.Read(buf[0:])
        result.Write(buf[0:n])
        if err != nil {
            if err == io.EOF {
                break
            }
            return nil, err
        }
    }
    return result.Bytes(), nil
}
```

执行这段程序并查看执行结果：

```
$ go build simplehttp.go
$ ./simplehttp qbox.me:80

HTTP/1.1 301 Moved Permanently
Server: nginx/1.0.14
Date: Mon, 21 May 2012 03:15:08 GMT
Content-Type: text/html
Content-Length: 184
Connection: close
Location: https://qbox.me/
```
