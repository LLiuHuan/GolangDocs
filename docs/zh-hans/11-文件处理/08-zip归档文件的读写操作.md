<center><h1>zip归档文件的读写操作</h1></center>

---

Go 语言的标准库提供了对几种压缩格式的支持，其中包括 gzip，因此 Go 程序可以无缝地读写 .gz 扩展名的 gzip 压缩文件或非 .gz 扩展名的非压缩文件。此外，标准库也提供了读和 写 .zip 文件、tar 包文件 (.tar 和 .tar.gz )，以及读 .bz2 文件（即 .tar .bz2 文件）的功能。

### 创建 zip 归档文件

要使用 Zip 包来压缩文件，我们首先必须打开一个用于写的文件，然后创建一个 zip.Writer 值来往其中写入数据。然后，对于每一个我们希望加入 .zip 归档文件的文件，我们必须读取该文件并将其内容写入 zip.Writer 中。该 pack 程序使用了 createZip() 和 writeFileToZip() 两个函数以这种方式来创建一个 .zip 文件。

```go
func createZip(filename string, files []string) error {
    file, err := os.Create(filename)
    if err != nil {
        return err
    }
    defer file.Close()
    zipper := zip.NewWriter(file)
    defer zipper.Close()
    for _, name := range files {
        if err := writeFileToZip(zipper, name); err != nil {
            return err
        }
    }
    return nil
}
```

该 createZip() 函数和 writeFileToZip() 函数都比较简短，因此容易让人觉得应该写入一个函数中。这是不明智的，因为在该 for 循环中我们可能打开一个又一个的文件（即 files 切片中的所有文件），从而可能超出操作系统允许的文件打开数上限。这点我们在前面章节中己有简短的阐述。

当然，我们可以在每次迭代中调用。s.File.Close()，而非延迟执行它，但这样做还必须保证程序无论是否出错文件都必须关闭。因此，最为简便而干净的解决方案是，像这里所做的那样，总是创建一个独立的函数来处理每个独立的文件。

```go
func writeFileToZip(zipper *zip.Writer, filename string) error {
    file, err := os.Open(filename)
    if err != nil {
        return err
    }
    defer file.Close()
    info, err := file.Stat()
    if err != nil {
        return err
    }
    header, err := zip.FilelnfoHeader(info)
    if err != nil {
        return err
    }
    header.name = sanitizedName(filename)
    writer, err := zipper.CreateHeader(header)
    if err != nil {
        return err
    }
    _, err = io.Copy(writer, file)
    return err
}
```

首先我们打开需要归档的文件以供读取，然后延迟关闭它。

接下来，我们调用 os.File.Stat() 方法来取得包含时间戳和权限标志的 os.Fileinfo 值。然后，我们将该值传给 zip.FileInfoHeader() 函数，该函数返回一个 zip.FileHeader 值，其中保存了时间戳、权限以及文件名。在压缩文件中，我们无需使用与原始文件名一样的文件名，因此这里我们使用净化过的文件名来覆盖原始文件名（保存在 zip.FileHeader.Name 字段中）。

头部设置好之后，我们将其作为参数调用 zip.CreateHeader() 函数。这会在 .zip 压缩文件中创建一个项，其中包含头部的时间戳、权限以及文件名，并返回一个 io.Writer，我们可以往其中写入需要被压缩的文件的内容。

为此，我们使用了 io.Copy() 函数，它能够返回所复制的字节数，以及一个为空或者非空的错误值。

如果在任何时候发生错误，该函数就会立即返回并由调用者处理错误。如果最终没有错误发生，那么该 .zip 压缩文件就会包含该给定文件。

```go
func sanitizedName(filename string) string{
    if len(filename) > 1 && filename[1] == ':' && runtime.GOOS == "windows" {
        filename = filename[2:]
    }
    filename = filepath.ToSlash(filename)
    filename = strings. TrimLef t( filename, "/.**)
    return strings.Replace(filename, "./",      -1)
}
```

如果一个归档文件中包含的文件带有绝对路径或者含有“..”路径组件，我们就有可能在解开归档的时候意外覆盖本地重要文件。为了降低这种风险，我们对保存在归档文件里每个文件的文件名都做了相应的净化。

该 sanitizedName() 函数会删除路径头部的盘符以及冒号（如果有的话），然后删除头部任何目录分隔符、点号以及任何路径组件，并将文件分隔符强制转换成正向斜线。

### 解开 zip 归档文件

解开一个 .zip 归档文件与创建一个归档文件一样简单，只是如果归档文件中包含带有路径的文件名，就必须重建目录结构。

```go
func unpackZip(filename string) error {
    reader, err := zip.OpenReader(filename)
    if err != nil {
        return err
    }
    defer reader.Close()
    for _, zipFile := range reader.Reader.File {
        name := san it izedName (zipFiJLe.Name)
        mode := zipFile.Mode()
        if mode.IsDir() {
            if err = os.MkdirAl1(name, 0755); err != nil {
                return err
            }
        } else {
            if err = unpackZippedFile(name, zipFile); err != nil {
                return err
            }
        }
    }
    return nil
}
```

该函数打开给定的 .zip 文件用于读取。这里没有使用 os.Open() 函数来打开文件后调用 zip.NewReader()，而是使用 zip 包提供的 zip.OpenReader() 函数，它可以方便地打开并返回一个 \*zip.ReadCloser 值让我们使用。

zip.ReadCloser 最为重要的一点是它包含了导出的 zip.Reader 结构体字段，其中包含一个包含指向 zip.File 结构体指针的 []\*zip.File 切片，其中的每一项表示 .zip 压缩文件中的一个文件。

我们迭代访问该 reader 的 zip.File 结构体，并创建一个净化过的文件及目录名（使用我们在 pack 程序中用到的 sanitizedName() 函数），以降低覆盖重要文件的风险。

如果遇到一个目录（由 \*zip.File 的 os.FileMode 的 IsDir() 方法报告），我们就创建一个目录。os.MkdirAll() 函数传入了有用的属性信息，会自动创建必要的中间目录以创建特定的目标目录，如果目录已经存在则会安全地返回 nil 而不执行任何操作。如果遇到的是一个文件，则交由自定义的 unpackZippedFile() 函数进行解压。

```go
func unpackZippedFile(filename string, zipFile *zipFile) error {
    writer, err := os.Create(filename)
    if err != nil {
        return err
    }
    defer writer.Close()
    reader, err := zipFile.Open()
    if err != nil {
        return err
    }
    defer reader.Close()
    if _, err = io.Copy(writer, reader); err != nil {
        return err
    }
    if filename == zipFile.Name {
        fm.Println(filename)
    } else {
        fmt.Printf("%s [%s]\n", filename, zipFile.Name)
    }
    return nil
}
```

unpackZippedFile() 函数的作用就是将 .zip 归档文件里的单个文件抽取出来，写到 filename 指定的文件里去。首先它创建所需要的文件，然后，使用 zip.File.Open() 函数打开指定的归档文件，并将数据复制到新创建的文件里去。

最后，如果没有错误发生，该函数会往终端打印所创建文件的文件名，如果处理后的文件名与原始文件名不一样，则将原始文件名包含在方括号中。

值得注意的是，该 \* zip.File 类型也有一些其他的方法，如 zip.File.Mode()（在前面的 unpackZip() 函数中已有使用），zip.File.ModTime()（以 time.Time 值返回文件的修改时间）以及返回文件的 os.Fileinfo 值的 zip.Fileinfo()。
