<center><h1>使用buffer读取文件</h1></center>

---

在下面的例子中，联合使用了缓冲读取文件和命令行 flag 解析这两项技术。如果不加参数，那么你输入什么屏幕就打印什么。

参数被认为是文件名，如果文件存在的话就打印文件内容到屏幕。命令行执行 cat test 测试输出。
```go
package main
import (
    "bufio"
    "flag"
    "fmt"
    "io"
    "os"
)
func cat(r *bufio.Reader) {
    for {
        buf, err := r.ReadBytes('\n')
        if err == io.EOF {
            break
        }
        fmt.Fprintf(os.Stdout, "%s", buf)
    }
    return
}
func main() {
    flag.Parse()
    if flag.NArg() == 0 {
        cat(bufio.NewReader(os.Stdin))
    }
    for i := 0; i < flag.NArg(); i++ {
        f, err := os.Open(flag.Arg(i))
        if err != nil {
            fmt.Fprintf(os.Stderr, "%s:error reading from %s: %s\n", os.Args[0], flag.Arg(i), err.Error())
            continue
        }
        cat(bufio.NewReader(f))
    }
}
```