<center><h1>range关键字</h1></center>

---

既然切片是一个集合，那么就可以迭代其中的元素。Go 语言有个特殊的关键字 range，它可以配合关键字 for 来迭代切片里的元素，如下所示：

```go
// 创建一个整型切片
// 其长度和容量都是 4 个元素
slice := []int{10, 20, 30, 40}
// 迭代每一个元素，并显示其值
for index, value := range slice {
    fmt.Printf("Index: %d Value: %d\n", index, value)
}
```

输出结果为：

```
Index: 0 Value: 10
Index: 1 Value: 20
Index: 2 Value: 30
Index: 3 Value: 40
```

当迭代切片时，关键字 range 会返回两个值。第一个值是当前迭代到的索引位置，第二个值是该位置对应元素值的一份副本，如下图所示。

<div align=center> 
    <img src="img/2-语言容器/07-range关键字/使用 range 迭代切片会创建每个元素的副本.gif"/> 
    <p>图：使用 range 迭代切片会创建每个元素的副本</p>
</div>

需要强调的是，range 创建了每个元素的副本，而不是直接返回对该元素的引用，如下所示。如果使用该值变量的地址作为指向每个元素的指针，就会造成错误。让我们看看是为什么。

【示例 1】range 提供了每个元素的副本

```go
// 创建一个整型切片
// 其长度和容量都是4 个元素
slice := []int{10, 20, 30, 40}
// 迭代每个元素，并显示值和地址
for index, value := range slice {
    fmt.Printf("Value: %d Value-Addr: %X ElemAddr: %X\n",
    value, &value, &slice[index])
}
```

输出结果为：

```
Value: 10 Value-Addr: 10500168 ElemAddr: 1052E100
Value: 20 Value-Addr: 10500168 ElemAddr: 1052E104
Value: 30 Value-Addr: 10500168 ElemAddr: 1052E108
Value: 40 Value-Addr: 10500168 ElemAddr: 1052E10C
```

因为迭代返回的变量是一个迭代过程中根据切片依次赋值的新变量，所以 value 的地址总是相同的。要想获取每个元素的地址，可以使用切片变量和索引值。

如果不需要索引值，可以使用占位字符来忽略这个值，代码如下所示。

【示例 2】使用空白标识符（下划线）来忽略索引值

```go
// 创建一个整型切片
// 其长度和容量都是 4 个元素
slice := []int{10, 20, 30, 40}
// 迭代每个元素，并显示其值
for _, value := range slice {
    fmt.Printf("Value: %d\n", value)
}
```

输出结果为：

```
Value: 10
Value: 20
Value: 30
Value: 40
```

关键字 range 总是会从切片头部开始迭代。如果想对迭代做更多的控制，依旧可以使用传统的 for 循环，代码如下所示。

【示例 3】使用传统的 for 循环对切片进行迭代

```go
// 创建一个整型切片
// 其长度和容量都是 4 个元素
slice := []int{10, 20, 30, 40}
// 从第三个元素开始迭代每个元素
for index := 2; index < len(slice); index++ {
    fmt.Printf("Index: %d Value: %d\n", index, slice[index])
}
```

输出结果为：

```
Index: 2 Value: 30
Index: 3 Value: 40
```
