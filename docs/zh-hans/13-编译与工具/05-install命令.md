<center><h1>install命令</h1></center>

---

### go install命令——编译并安装

go install 命令的功能和前面一节《[go build命令](zh-hans/13-编译与工具/01-build命令)》中介绍的 go build 命令类似，附加参数绝大多数都可以与 go build 通用。go install 只是将编译的中间文件放在 GOPATH 的 pkg 目录下，以及固定地将编译结果放在 GOPATH 的 bin 目录下。

这个命令在内部实际上分成了两步操作：第一步是生成结果文件（可执行文件或者 .a 包），第二步会把编译好的结果移到 $GOPATH/pkg 或者 $GOPATH/bin。

使用 go install 来执行代码，参考下面的 shell：

```go
$ export GOPATH=/home/davy/golangbook/code
$ go install chapter11/goinstall
```

编译完成后的目录结构如下：

```go
.
├── bin
│   └── goinstall
├── pkg
│   └── linux_amd64
│       └── chapter11
│           └── goinstall
│               └── mypkg.a
└── src
    └── chapter11
        ├── gobuild
        │   ├── lib.go
        │   └── main.go
        └── goinstall
            ├── main.go
            └── mypkg
                └── mypkg.go
```

go install 的编译过程有如下规律：

- go install 是建立在 GOPATH 上的，无法在独立的目录里使用 go install。
- GOPATH 下的 bin 目录放置的是使用 go install 生成的可执行文件，可执行文件的名称来自于编译时的包名。
- go install 输出目录始终为 GOPATH 下的 bin 目录，无法使用`-o`附加参数进行自定义。
- GOPATH 下的 pkg 目录放置的是编译期间的中间文件。

