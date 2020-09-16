<center><h1>switch语句</h1></center>

---

Go 语言要比 C 语言的更加通用。表达式不需要为常量，甚至不需要为整数，case 是按照从上到下的顺序进行求值，直到找到匹配的。如果 switch 没有表达式，则对 true 进行匹配。因此，可以按照语言习惯将 if-else-if-else 链写成一个 switch。

相比较 C 语言和 Java 等其它语言而言，Go 语言中的 switch 结构使用上更加灵活，语法设计尽量以使用方便为主。

### 基本写法

Go 语言改进了 switch 的语法设计，避免人为造成失误。Go 语言的 switch 中的每一个 case 与 case 间是独立的代码块，不需要通过 break 语句跳出当前 case 代码块以避免执行到下一行。示例代码如下：

```go
var a = "hello"
switch a {
case "hello":
    fmt.Println(1)
case "world":
    fmt.Println(2)
default:
    fmt.Println(0)
}
```

代码输出如下：

```
1
```

上面例子中，每一个 case 均是字符串格式，且使用了 default 分支，Go 语言规定每个 switch 只能有一个 default 分支。

**1) 一分支多值**

当出现多个 case 要放在一起的时候，可以像下面代码这样写：

```go
var a = "mum"
switch a {
case "mum", "daddy":
    fmt.Println("family")
}
```

不同的 case 表达式使用逗号分隔。

**2) 分支表达式**

case 后不仅仅只是常量，还可以和 if 一样添加表达式，代码如下：

```go
var r int = 11
switch {
case r > 10 && r < 20:
    fmt.Println(r)
}
```

> 注意，这种情况的 switch 后面不再跟判断变量，连判断的目标都没有了。

### 跨越 case 的 fallthrough——兼容 C 语言的 case 设计

在 Go 语言中 case 是一个独立的代码块，执行完毕后不会像 C 语言那样紧接着下一个 case 执行。但是为了兼容一些移植代码，依然加入了 fallthrough 关键字来实现这一功能，代码如下：

```go
var s = "hello"
switch {
case s == "hello":
    fmt.Println("hello")
    fallthrough
case s != "world":
    fmt.Println("world")
}
```

代码输出如下：

```
hello

world
```

> 新编写的代码，不建议使用 fallthrough。

### 类型 switch

switch 还可用于获得一个接口变量的动态类型。这种类型 switch 使用类型断言的语法，在括号中使用关键字 type 。如果 switch 在表达式中声明了一个变量，则变量会在每个子句中具有对应的类型。比较符合控制结构语言习惯的方式是在这些 case 里重用一个名字，实际上是在每个 case 里声名一个新的变量，其具有相同的名字，但是不同的类型。

```go
var t interface{}
t = functionOfSomeType()
switch t := t.(type) {
default:
    fmt.Printf("unexpected type %T", t)       // %T 打印任何类型的t
case bool:
    fmt.Printf("boolean %t\n", t)             // t 的类型是 bool
case int:
    fmt.Printf("integer %d\n", t)             // t 的类型是 int
case *bool:
    fmt.Printf("pointer to boolean %t\n", *t) // t 的类型是 *bool
case *int:
    fmt.Printf("pointer to integer %d\n", *t) // t 的类型是 *int
}
```
