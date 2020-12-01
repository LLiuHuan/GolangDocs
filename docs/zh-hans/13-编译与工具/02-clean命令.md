<center><h1>clean命令</h1></center>

---

## go clean命令——清除编译文件

Go语言中 go clean 命令是用来移除当前源码包和关联源码包里面编译生成的文件。这些文件包括：

- _obj/ 旧的 object 目录，由 Makefiles 遗留
- _test/ 旧的 test 目录，由 Makefiles 遗留
- _testmain.go 旧的 gotest 文件，由M akefiles 遗留
- test.out 旧的 test 记录，由 Makefiles 遗留
- build.out 旧的 test 记录，由 Makefiles 遗留
- *.[568ao] object 文件，由 Makefiles 遗留
- DIR(.exe) 由 go build 产生
- DIR.test(.exe) 由 go test -c 产生
- MAINFILE(.exe) 由 go build MAINFILE.go 产生
- *.so 由 SWIG 产生

我一般都是利用这个命令清除编译文件，然后 github 递交源码，在本机测试的时候这些编译文件都是和系统相关的，但是对于源码管理来说没必要。

```go
$ go clean -i -n
cd /Users/astaxie/develop/gopath/src/mathapp
rm -f mathapp mathapp.exe mathapp.test mathapp.test.exe app app.exe
rm -f /Users/astaxie/develop/gopath/bin/mathapp
```

参数介绍

- -i 清除关联的安装的包和可运行文件，也就是通过 go install 安装的文件
- -n 把需要执行的清除命令打印出来，但是不执行，这样就可以很容易的知道底层是如何运行的
- -r 循环的清除在 import 中引入的包
- -x 打印出来执行的详细命令，其实就是 -n 打印的执行版本

