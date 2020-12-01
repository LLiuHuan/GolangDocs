<center><h1>generate命令</h1></center>

---

### go generate命令——在编译前自动化生成某类代码

go generate 命令是从 Go1.4 开始才设计的，用于在编译前自动化生成某类代码。 go generate 和 go build 是完全不一样的命令，通过分析源码中特殊的注释，然后执行相应的命令。

这些命令都是很明确的，没有任何的依赖在里面。而且大家在用这个之前心里面一定要有一个理念，这个 go generate 是给你用的，不是给使用你这个包的人用的，是方便你来生成一些代码的。

有几点需要注意：

- 该特殊注释必须在 .go 源码文件中。
- 每个源码文件可以包含多个 generate 特殊注释时。
- 显示运行 go generate 命令时，才会执行特殊注释后面的命令。
- 命令串行执行的，如果出错，就终止后面的执行。
- 特殊注释必须以"//go:generate"开头，双斜线后面没有空格。

这里我们来举一个简单的例子，例如我们经常会使用 yacc 来生成代码，那么我们常用这样的命令：

```go
go tool yacc -o gopher.go -p parser gopher.y
```

-o 指定了输出的文件名， -p 指定了 package 的名称，这是一个单独的命令，如果我们想让 go generate 来触发这个命令，那么就可以在当然目录的任意一个 xxx.go 文件里面的任意位置增加一行如下的注释：

```go
//go:generate go tool yacc -o gopher.go -p parser gopher.y
```

这里我们注意了， //go:generate 是没有任何空格的，这其实就是一个固定的格式，在扫描源码文件的时候就是根据这个来判断的。

所以我们可以通过如下的命令来生成，编译，测试。如果 gopher.y 文件有修改，那么就重新执行 go generate 重新生成文件就好。

```go
$ go generate
$ go build
$ go test
```

