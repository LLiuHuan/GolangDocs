<center><h1>fmt命令</h1></center>

---

### go fmt命令——格式化代码文件

在Go语言中，代码则有标准的风格。由于之前已经有的一些习惯或其它的原因我们常将代码写成 ANSI 风格或者其它更合适自己的格式，这将为人们在阅读别人的代码时添加不必要的负担，所以 go 强制了代码格式（比如左大括号必须放在行尾），不按照此格式的代码将不能编译通过。

为了减少浪费在排版上的时间，go 工具集中提供了一个 go fmt 命令它可以帮你格式化你写好的代码文件，使你写代码的时候不需要关心格式，只需要在写完之后执行`go fmt <文件名>.go` ，代码就会被修改成了标准格式。

但是平常很少用到这个命令，因为开发工具里面一般都带了保存时候自动格式化功能，这个功能其实在底层就是调用了 go fmt。

接下来的一节我将讲述两个工具，这两个工具都自带了保存文件时自动化 go fmt 功能。

使用 go fmt 命令，其实是调用了 gofmt，而且需要参数 -w，否则格式化结果不会写入文件。`gofmt -w -l src`，可以格式化整个项目。所以 go fmt 是 gofmt 的上层一个包装的命令。

gofmt 的参数介绍

- -l 显示那些需要格式化的文件
- -w 把改写后的内容直接写入到文件中，而不是作为结果打印到标准输出。
- -r 添加形如“a[b:len(a)] -> a[b:]”的重写规则，方便我们做批量替换
- -s 简化文件中的代码
- -d 显示格式化前后的 diff 而不是写入文件，默认是 false
- -e 打印所有的语法错误到标准输出。如果不使用此标记，则只会打印不同行的前 10 个错误。
- -cpuprofile 支持调试模式，写入相应的 cpufile 到指定的文件