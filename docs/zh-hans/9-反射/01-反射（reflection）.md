<center><h1>反射（reflection）</h1></center>

---

Go 语言提供了一种机制在运行时更新变量和检查它们的值、调用它们的方法和它们支持的内在操作，但是在编译时并不知道这些变量的具体类型。这种机制被称为反射。反射也可以让我们将类型本身作为第一类的值类型处理。

反射（reflection）是在 Java 出现后迅速流行起来的一种概念。通过反射，你可以获取丰富的类型信息，并可以利用这些类型信息做非常灵活的工作。

在 Java 中，你可以读取配置并根据类型名称创建对应的类型，这是一种常见的编程手法。Java 中的很多重要框架和技术（比如 Spring/IoC、Hibernate、Struts）等都严重依赖于反射功能。虽然，使用 Java EE 时很多人都觉得很麻烦，比如需要配置大量 XML 格式的配置程序，但这毕竟不是反射的错，反而更加说明了反射所带来的高可配置性。

大多数现代的高级语言都以各种形式支持反射功能，除了一切以性能为上的 C++语言。Go 语言的反射实现了反射的大部分功能，但没有像 Java 语言那样内置类型工厂，故而无法做到像 Java 那样通过类型字符串创建对象实例。

反射是把双刃剑，功能强大但代码可读性并不理想。若非必要，并不推荐使用反射，下面我们将介绍反射功能在 Go 语言中的具体体现以及反射的基本使用方法。

### 基本概念

Go 语言中的反射与其他语言有比较大的不同。首先要理解两个基本概念 Type 和 Value，它们也是 Go 语言包中 reflect 空间里最重要的两个类型。先看一下下面的定义：

```go
type MyReader struct {
    Name string
}
func (r MyReader)Read(p []byte) (n int, err error) {
    // 实现自己的Read方法
}
```

因为 MyReader 类型实现了 io.Reader 接口的所有方法（其实就是一个 Read() 函数），所以 MyReader 实现了接口 io.Reader。可以按如下方式来进行 MyReader 的实例化和赋值：

```go
var reader io.Reader
reader = &MyReader{"a.txt"}
```

对所有接口进行反射，都可以得到一个包含 Type 和 Value 的信息结构。比如对上例的 reader 进行反射，也将得到一个 Type 和 Value，Type 为 io.Reader，Value 为 MyReader{"a.txt"}。顾名思义，Type 主要表达的是被反射的这个变量本身的类型信息，而 Value 则为该变量实例本身的信息。

### 基本用法

通过使用 Type 和 Value，可以对一个类型进行各项灵活的操作。接下来分别演示反射的几种最基本用途。

#### 1) 获取类型信息

为了理解反射的功能，先来看看下面代码所示的这个小程序。

```go
package main
import (
    "fmt"
    "reflect"
)
func main() {
    var x float64 = 3.4
    fmt.Println("type:", reflect.TypeOf(x))
}
```

运行上述代码，输出结果如下所示：

```
type: float64
```

Type 和 Value 都包含了大量的方法，其中第一个有用的方法应该是 Kind，这个方法返回该类型的具体信息：Uint、Float64 等。Value 类型还包含了一系列类型方法，比如 Int()，用于返回对应的值。查看以下示例：

```go
var x float64 = 3.4
v := reflect.ValueOf(x)
fmt.Println("type:", v.Type())
fmt.Println("kind is float64:", v.Kind() == reflect.Float64)
fmt.Println("value:", v.Float())
```

结果为：

```
type: float64
kind is float64: true
value: 3.4
```

#### 2) 获取值类型

类型 Type 中有一个成员函数 CanSet()，其返回值为 bool 类型。如果你在注意到这个函数之前就直接设置了值，很有可能会收到一些看起来像异常的错误处理消息。

可能很多人会置疑为什么要有这么个奇怪的函数，可以设置所有的域不是很好吗？这里先解释一下这个函数存在的原因。

在之前的学习中提到过 Go 语言中所有的类型都是值类型，即这些变量在传递给函数的时候将发生一次复制。基于这个原则，我们再次看一下下面的语句：

```go
var x float64 = 3.4
v := reflect.ValueOf(x)
v.Set(4.1)
```

最后一条语句试图修改 v 的内容。是否可以成功地将 x 的值改为 4.1 呢？先要理清 v 和 x 的关系。在调用 ValueOf() 的地方，需要注意到 x 将会产生一个副本，因此 ValueOf() 内部对 x 的操作其实都是对着 x 的一个副本。

假如 v 允许调用 Set()，那么我们也可以想象出，被修改的将是这个 x 的副本，而不是 x 本身。如果允许这样的行为，那么执行结果将会非常困惑。调用明明成功了，为什么 x 的值还是原来的呢？为了解决这个问题 Go 语言，引入了可设属性这个概念（Settability）。如果 CanSet() 返回 false，表示你不应该调用 Set() 和 SetXxx() 方法，否则会收到这样的错误：

```
panic: reflect.Value.SetFloat using unaddressable value
```

现在知道，有些场景下不能使用反射修改值，那么到底什么情况下可以修改的呢？其实这还是跟传值的道理类似。我们知道，直接传递一个 float 到函数时，函数不能对外部的这个 float 变量有任何影响，要想有影响的话，可以传入该 float 变量的指针。下面的示例小幅修改了之前的例子，成功地用反射的方式修改了变量 x 的值：

```go
var x float64 = 3.4
p := reflect.ValueOf(&x) // 注意：得到X的地址
fmt.Println("type of p:", p.Type())
fmt.Println("settability of p:" , p.CanSet())
v := p.Elem()
fmt.Println("settability of v:" , v.CanSet())
v.SetFloat(7.1)
fmt.Println(v.Interface())
fmt.Println(x)
```

### 对结构的反射操作

之前讨论的都是简单类型的反射操作，现在讨论一下结构的反射操作。下面的示例演示了如何获取一个结构中所有成员的值：

```go
type T struct {
    A int
    B string
}
t := T{203, "mh203"}
s := reflect.ValueOf(&t).Elem()
typeOfT := s.Type()
for i := 0; i < s.NumField(); i++ {
    f := s.Field(i)
    fmt.Printf("%d: %s %s = %v\n", i,
        typeOfT.Field(i).Name, f.Type(), f.Interface())
}
```

以上例子的输出为：

```
0: A int = 203
1: B string = mh203
```

可以看出，对于结构的反射操作并没有根本上的不同，只是用了 Field() 方法来按索引获取对应的成员。当然，在试图修改成员的值时，也需要注意可赋值属性。
