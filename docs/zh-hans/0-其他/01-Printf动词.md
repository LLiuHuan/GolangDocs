<center><h1>Printf动词</h1></center>

---

| 动词 | 功能                                                         |
| ---- | ------------------------------------------------------------ |
| %v   | 按值本身的格式输出，万能动词，不知道用什么动词就用它         |
| %+v  | 同%v，但输出结构体时会添加字段名                             |
| %#v  | 输出值得Go语法表示                                           |
| %t   | 格式化`bool`值                                               |
| %b   | 按二进制输出                                                 |
| %c   | 输出整数对应的Unicode字符                                    |
| %d   | 按十进制输出                                                 |
| %o   | 按八进制输出                                                 |
| %O   | 按八进制输出，并添加`0o`前缀                                 |
| %x   | 按十六进制输出，字母小写，%x还能用来格式化字符串和`[]byte`类型，每个字节用两位十六进制表示，字母小写 |
| %X   | 按十六进制输出，字母大写，%X还能用来格式化字符串和`[]byte`类型，每个字节用两位十六进制表示，字母小写 |
| %U   | 按Unicode格式输出                                            |
| %e   | 按科学计数法表示输出，字母小写                               |
| %E   | 按科学计数法表示输出，字母大写                               |
| %f   | 输出浮点数                                                   |
| %F   | 同%f                                                         |
| %g   | 漂亮的格式化浮点数                                           |
| %G   | 同%G                                                         |
| %s   | 格式化为字符串                                               |
| %q   | 格式化为字符串，并在两端添加双引号                           |
| %p   | 格式化指针                                                   |
| %T   | 输出变量类型                                                 |
| %w   | 专用于`fmt.Errorf`包装`error`                                |

示例：

```go
package main

import "fmt"

type point struct {
	x, y, z int
}

var (
	p = point{1, 2, 3}
)

func void main() {
	//{1 2 3}
	fmt.Printf("%v\n", p)
	
	//{x:1 y:2 z:3}
	fmt.Printf("%+v\n", p)
	
	//main.point{x:1, y:2, z:3}
	fmt.Printf("%#v\n", p)
	
	//true
	fmt.Printf("%t\n", true)
	
	//1010
	fmt.Printf("%b\n", 10)
	
	//A
	fmt.Printf("%c\n", 65)
	
	//1
	fmt.Printf("%d\n", 1)
	
	//11
	fmt.Printf("%o\n", 9)
	
	//0o11
	fmt.Printf("%O\n", 9)
	
	//a
	fmt.Printf("%x\n", 10)
	
	//A
	fmt.Printf("%X\n", 10)
	
	//010a
	fmt.Printf("%x\n", []byte{1, 10})
	
	//010A
	fmt.Printf("%X\n", []byte{1, 10})
	
	//68656c6c6f
	fmt.Printf("%x\n", "hello")
	
	//776F726C64
	fmt.Printf("%X\n", "world")
	
	//U+0001
	fmt.Printf("%U\n", 1)
	
	//1.200000e+00
	fmt.Printf("%e\n", 1.2)
	
	//1.200000E+00
	fmt.Printf("%E\n", 1.2)
	
	//1.200000
	fmt.Printf("%f\n", 1.2)
	
	//1.200000
	fmt.Printf("%F\n", 1.2)
	
	//1.2
	fmt.Printf("%g\n", 1.2)
	
	//1.2
	fmt.Printf("%G\n", 1.2)
	
	//hello
	fmt.Printf("%s\n", "hello")
	
	//"world"
	fmt.Printf("%q\n", "world")
	
	//0x563820
	fmt.Printf("%p\n", &p)
	
	//main.point
	fmt.Printf("%T\n", p)
	
	//outer:inner
	fmt.Println(fmt.Errorf("outer:%w", errors.New("inner")))
}
```