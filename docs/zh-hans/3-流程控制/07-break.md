<center><h1>break（跳出循环）</h1></center>

---

Go 语言中，一个 break 的作用范围为该语句出现后的最内部的结构，它可以被用于任何形式的 for 循环（计数器、条件判断等）。但在 switch 或 select 语句中，break 语句的作用结果是跳过整个代码块，执行后续的代码。

循环嵌套循环时，可以在 break 后指定标签。用标签决定哪个循环被终止。

跳出指定循环：

```go
package main
import "fmt"
func main() {
OuterLoop:
    for i := 0; i < 2; i++ {
        for j := 0; j < 5; j++ {
            switch j {
            case 2:
                fmt.Println(i, j)
                break OuterLoop
            case 3:
                fmt.Println(i, j)
                break OuterLoop
            }
        }
    }
}
```

代码输出如下：

```
0 2
```

代码说明如下：

```
第 7 行，外层循环的标签。
第 8 行和第 9 行，双层循环。
第 10 行，使用 switch 进行数值分支判断。
第 13 和第 16 行，退出 OuterLoop 对应的循环之外，也就是跳转到第 20 行。
```
