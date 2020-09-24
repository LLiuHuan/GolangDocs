<center><h1>解码未知结构的JSON数据</h1></center>

---

Go 语言支持接口。在 Go 语言里，接口是一组预定义方法的组合，任何一个类型均可通过实现接口预定义的方法来实现，且无需显示声明，所以没有任何方法的空接口可以代表任何类型。换句话说，每一个类型其实都至少实现了一个空接口。

Go 内建这样灵活的类型系统，向我们传达了一个很有价值的信息：空接口是通用类型。如果要解码一段未知结构的 JSON，只需将这段 JSON 数据解码输出到一个空接口即可。

在解码 JSON 数据的过程中，JSON 数据里边的元素类型将做如下转换：

```
JSON 中的布尔值将会转换为 Go 中的 bool 类型；
数值会被转换为 Go 中的 float64 类型；
字符串转换后还是 string 类型；
JSON 数组会转换为 []interface{} 类型；
JSON 对象会转换为 map[string]interface{} 类型；
null 值会转换为 nil。
```

在 Go 的标准库 encoding/json 包中，允许使用 map[string]interface{} 和 []interface{} 类型的值来分别存放未知结构的 JSON 对象或数组，示例代码如下：

```go
b := []byte(`{
    "Title": "Go语言编程",
    "Authors": ["XuShiwei", "HughLv", "Pandaman", "GuaguaSong", "HanTuo", "BertYuan",
    "XuDaoli"],
    "Publisher": "ituring.com.cn",
    "IsPublished": true,
    "Price": 9.99,
    "Sales": 1000000
}`)
var r interface{}
err := json.Unmarshal(b, &r)
```

在上述代码中，r 被定义为一个空接口。json.Unmarshal() 函数将一个 JSON 对象解码到空接口 r 中，最终 r 将会是一个键值对的 map[string]interface{} 结构：

```go
map[string]interface{}{
    "Title": "Go语言编程",
    "Authors": ["XuShiwei", "HughLv", "Pandaman", "GuaguaSong", "HanTuo", "BertYuan",
    "XuDaoli"],
    "Publisher": "ituring.com.cn",
    "IsPublished": true,
    "Price": 9.99,
    "Sales": 1000000
}
```

要访问解码后的数据结构，需要先判断目标结构是否为预期的数据类型：

```go
gobook, ok := r.(map[string]interface{})
```

然后，我们可以通过 for 循环搭配 range 语句一一访问解码后的目标数据：

```go
if ok {
    for k, v := range gobook {
        switch v2 := v.(type) {
            case string:
                fmt.Println(k, "is string", v2)
            case int:
                fmt.Println(k, "is int", v2)
            case bool:
                fmt.Println(k, "is bool", v2)
            case []interface{}:
                fmt.Println(k, "is an array:")
                for i, iv := range v2 {
                    fmt.Println(i, iv)
                }
            default:
                fmt.Println(k, "is another type not handle yet")
        }
    }
}
```

虽然有些烦琐，但的确是一种解码未知结构的 JSON 数据的安全方式。

### JSON 的流式读写

Go 内建的 encoding/json 包还提供 Decoder 和 Encoder 两个类型，用于支持 JSON 数据的流式读写，并提供 NewDecoder() 和 NewEncoder() 两个函数来便于具体实现：

```go
func NewDecoder(r io.Reader) *Decoder
func NewEncoder(w io.Writer) *Encoder
```

下面代码演示了从标准输入流中读取 JSON 数据，然后将其解码，但只保留 Title 字段（书名），再写入到标准输出流中。

```go
package main
import (
    "encoding/json"
    "log"
    "os"
)
func main() {
    dec := json.NewDecoder(os.Stdin)
    enc := json.NewEncoder(os.Stdout)
    for {
        var v map[string]interface{}
        if err := dec.Decode(&v); err != nil {
            log.Println(err)
            return
        }
        for k := range v {
            if k != "Title" {
                v[k] = nil, false
            }
        }
        if err := enc.Encode(&v); err != nil {
            log.Println(err)
        }
    }
}
```

使用 Decoder 和 Encoder 对数据流进行处理可以应用得更为广泛些，比如读写 HTTP 连接、WebSocket 或文件等，Go 的标准库 net/rpc/jsonrpc 就是一个应用了 Decoder 和 Encoder 的实际例子。
