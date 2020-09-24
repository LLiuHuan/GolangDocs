<center><h1>DialTCP()</h1></center>

---

Dial() 函数其实是对 DialTCP()、DialUDP()、DialIP() 和 DialUnix() 的封装。我们也可以直接调用这些函数，它们的功能是一致的。这些函数的原型如下：

```go
func DialTCP(net string, laddr, raddr *TCPAddr) (c *TCPConn, err error)
func DialUDP(net string, laddr, raddr *UDPAddr) (c *UDPConn, err error)
func DialIP(netProto string, laddr, raddr *IPAddr) (*IPConn, error)
func DialUnix(net string, laddr, raddr *UnixAddr) (c *UnixConn, err error)
```

之前基于 TCP 发送 HTTP 请求，读取服务器返回的 HTTP Head 的整个流程也可以使用下面代码所示的实现方式。

```go
package main
import (
    "net"
    "os"
    "fmt"
    "io/ioutil"
)
func main() {
    if len(os.Args) != 2 {
    fmt.Fprintf(os.Stderr, "Usage: %s host:port", os.Args[0])
    os.Exit(1)
    }
    service := os.Args[1]
    tcpAddr, err := net.ResolveTCPAddr("tcp4", service)
    checkError(err)
    conn, err := net.DialTCP("tcp", nil, tcpAddr)
    checkError(err)
    _, err = conn.Write([]byte("HEAD / HTTP/1.0\r\n\r\n"))
    checkError(err)
    result, err := ioutil.ReadAll(conn)
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
```

与之前使用 Dail() 的例子相比，这里有两个不同：

```
net.ResolveTCPAddr()，用于解析地址和端口号；
net.DialTCP()，用于建立链接。
```

这两个函数在 Dial() 中都得到了封装。

此外，net 包中还包含了一系列的工具函数，合理地使用这些函数可以更好地保障程序的质量。

验证 IP 地址有效性的代码如下：

```go
func net.ParseIP()
```

创建子网掩码的代码如下：

```go
func IPv4Mask(a, b, c, d byte) IPMask
```

获取默认子网掩码的代码如下：

```go
func (ip IP) DefaultMask() IPMask
```

根据域名查找 IP 的代码如下：

```go
func ResolveIPAddr(net, addr string) (*IPAddr, error)
func LookupHost(name string) (cname string, addrs []string, err error)；
```
