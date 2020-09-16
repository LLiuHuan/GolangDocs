<center><h1>nil：空值/零</h1></center>

---

在不同语言中，表示空这个概念都有细微不同。比如在 scheme 语言（一种 lisp 方言）中，nil 是 true 的，而在 ruby 语言中，一切都是对象，连 nil 也是—个对象！在 C 中 NULL 跟 0 是等价的。

按照 Go 语言规范，任何类型在未初始化时都对应一个零值：布尔类型是 false，整型是 0，字符串是 ""，而指针，函数，interface，slice，channel 和 map 的零值都是 nil。

### interface

一个 interface 在没有进行初始化时，对应的值是 nil。也就是说 var v interface{}，此时 v 就是—个 nil。在底层存储上，它是—个空指针。与之不同的情况是，interface 值为空。比如：

```go
var v *T
var i interface{}
i = v
```

此时 i 是一个 interface，它的值是 nil，但它自身不为 nil。

Go 语言中的 error 其实就是一个实现了 Error 方法的接口：

```go
type error interface {
    Error() string
}
```

因此，我们可以自定义一个 error：

```go
type Error struct {
        errCode uint8
}
func (e *Error) Error() string {
    switch e.errCode {
    case 1:
        return "file not found"
    case 2:
        return "time out"
    case 3:
        return "permission denied"
    default:
        return "unknown error"
    }
}
```

如果我们这样使用它：

```go
func checkError(err error) {
        if err != nil {
        panic(err)
    }
}
var e *Error
checkError(e)
```

总之，interface 跟 C 语言的指针一样非常灵活，关于空的语义，也跟空指针一样容易困扰新手的，需要注意。

### string 和 slice

string 的空值是 ""，它是不能跟 nil 比较的。即使是空的 string，它的大小也是两个机器字长的。slice 也类似，它的空值并不是一个空指针，而是结构体中的指针域为空，空的 slice 的大小也是三个机器字长的。

### channel 和 map

channel 跟 string 或 slice 有些不同，它在栈上只是—个指针，实际的数据都是由指针所指向的堆上面。

跟 channel 相关的操作有：初始化/读/写/关闭。channel 未初始化值就是 nil，未初始化的 channel 是不能使用的。下面是—些操作规则：

```
读或者写一个 nil 的 channel 的操作会永远阻塞。
读一个关闭的 channel 会立刻返回一个 channel 元索类型的零值。
写一个关闭的 channel 会导致 panic。
```

map 也是指针，实际数据在堆中，未初始化的值是 nil。
