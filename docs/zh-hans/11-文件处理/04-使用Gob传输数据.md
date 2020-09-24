<center><h1>使用Gob传输数据</h1></center>

---

Gob 是 Go 语言自己的以二进制形式序列化和反序列化程序数据的格式；可以在 encoding 包中找到。这种格式的数据简称为 Gob （即 Go binary 的缩写）。类似于 Python 的 "pickle" 和 Java 的"Serialization"。

Gob 通常用于远程方法调用参数和结果的传输，以及应用程序和机器之间的数据传输。它和 JSON 或 XML 有什么不同呢？Gob 特定地用于纯 Go 的环境中，例如，两个用 Go 写的服务之间的通信。这样的话服务可以被实现得更加高效和优化。

Gob 不是可外部定义，语言无关的编码方式。因此它的首选格式是二进制，而不是像 JSON 和 XML 那样的文本格式。Gob 并不是一种不同于 Go 的语言，而是在编码和解码过程中用到了 Go 的反射。

Gob 文件或流是完全自描述的：里面包含的所有类型都有一个对应的描述，并且总是可以用 Go 解码，而不需要了解文件的内容。

只有可导出的字段会被编码，零值会被忽略。在解码结构体的时候，只有同时匹配名称和可兼容类型的字段才会被解码。当源数据类型增加新字段后，Gob 解码客户端仍然可以以这种方式正常工作：解码客户端会继续识别以前存在的字段。并且还提供了很大的灵活性，比如在发送者看来，整数被编码成没有固定长度的可变长度，而忽略具体的 Go 类型。

假如在发送者这边有一个有结构 T：

```go
type T struct { X, Y, Z int }
var t = T{X: 7, Y: 0, Z: 8}
```

而在接收者这边可以用一个结构体 U 类型的变量 u 来接收这个值：

```go
type U struct { X, Y *int8 }
var u U
```

在接收者中，X 的值是 7，Y 的值是 0（Y 的值并没有从 t 中传递过来，因为它是零值）和 JSON 的使用方式一样，Gob 使用通用的 io.Writer 接口，通过 NewEncoder() 函数创建 Encoder 对象并调用 Encode()；相反的过程使用通用的 io.Reader 接口，通过 NewDecoder() 函数创建 Decoder 对象并调用 Decode 。

【示例 1】下面代码是一个编解码，并且以字节缓冲模拟网络传输的简单例子：

```go
package main
import (
    "bytes"
    "fmt"
    "encoding/gob"
    "log"
)
type P struct {
    X, Y, Z int
    Name string
}
type Q struct {
    X, Y *int32
    Name string
}
func main() {
    //初始化编码器和解码器。通常 ENC 和 DEC 为
    //绑定到网络连接，编码器和解码器将
    //在不同的进程中运行。
    var network bytes.Buffer // 网络连接
    enc := gob.NewEncoder(&network) // 写入网络
    dec := gob.NewDecoder(&network) // 从网络读取
    // 对值进行编码（发送）
    err := enc.Encode(P{3, 4, 5, "Pythagoras"})
    if err != nil {
        log.Fatal("encode error:", err)
    }
    // 解码（接收）值
    var q Q
    err = dec.Decode(&q)
    if err != nil {
        log.Fatal("decode error:", err)
    }
    fmt.Printf("%q: {%d,%d}\n", q.Name, *q.X, *q.Y)
}
// Output: "Pythagoras": {3,4}
```

【示例 2】编码到文件：

```go
package main
import (
    "encoding/gob"
    "log"
    "os"
)
type Address struct {
    Type string
    City string
    Country string
}
type VCard struct {
    FirstName string
    LastName string
    Addresses []*Address
    Remark string
}
var content string
func main() {
    pa := &Address{"private", "Aartselaar","Belgium"}
    wa := &Address{"work", "Boom", "Belgium"}
    vc := VCard{"Jan", "Kersschot", []*Address{pa,wa}, "none"}
    // fmt.Printf("%v: \n", vc) // {Jan Kersschot [0x126d2b80 0x126d2be0] none}:
    // 使用编码器
    file, _ := os.OpenFile("vcard.gob", os.O_CREATE|os.O_WRONLY, 0)
    defer file.Close()
    enc := gob.NewEncoder(file)
    err := enc.Encode(vc)
    if err != nil {
        log.Println("Error in encoding gob")
    }
}
```
