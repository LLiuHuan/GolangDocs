<center><h1>示例：使用range复用临时变量</h1></center>

---

在开始本节的讲解之前，大家先来看一段简单的代码：

```go
package main
import "sync"
func main () {
    wg := sync.WaitGroup{}
    si := []int{l, 2, 3, 4, 5, 6, 7, 8, 9, 10}
    for i := range si {
        wg.Add (i)
        go func () {
            println(i)
            wg.Done()
        }()
    }
    wg.Wait()
}
```

运行结果：

```
9
9
9
9
9
9
9
9
9
9
```

程序结果并没有如我们预期一样遍历切片白，而是全部打印 9，有两点原因导致这个问题：

```
for range 下的迭代变量 i 的值是共用的。
main 函数所在的 goroutine 和后续启动的 goroutines 存在竞争关系。
```

使用 `go run -race` 来看一下数据竞争情况：

```
#CGO ENABLED=l go run - race src/c7_2_la.go
WARNING: DATA RACE
Read at 0x00c4200140b8 by goroutine 13:
    main.main.funcl()
        /project/go/src/gitbook/gobook/chapter7/src/c7_2_la.go:14 +0x38
Previous write at 0x00c4200140b8 by main goroutine:
    main.main ()
        /project/go/src/gitbook/gobook/chapter7/src/c7_2_la.go:11 +0xdf
Goroutine 13 (running) created at:
    main.main ()
        /project/go/src/gitbook/gobook/chapter7/src/c7_2_la.go:l3 +0xl35
=================
9
9
9
9
9
9
9
9
9
9
Found 1 data race(s)
exit status 66
```

可以看到 Goroutine 13 和 main goroutine 存在数据竞争，更进一步证实了 range 共享临时变量。range 在迭代写的过程中，多个 goroutine 并发地去读。

正确的写法是使用函数参数做一次数据复制，而不是闭包。示例如下：

```
package main
import "sync"
func main () {
    wg := sync.WaitGroup{}
    si := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
    for i := range si {
        wg.Add(1)
        //这里有一个实参到形参的值拷贝
        go func(a int) {
            println(a)
            wg.Done()
        }(i)
    }
    wg.Wait ()
}
```

运行结果：

```
9
0
1
2
3
4
5
6
7
8
```

可以看到新程序的运行结果符合预期。这个不能说是缺陷，而是 Go 语言设计者为了性能而选择的一种设计方案，因为大多情况下 for 循环块里的代码是在同一个 goroutine 里运行的，为了避免空间的浪费和 GC 的压力，复用了 range 迭代临时变量。语言使用者明白这个规约，在 for 循环下调用并发时要复制造代变量后再使用，不要直接引用 for 迭代变量。
