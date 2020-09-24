<center><h1>tar归档文件的读写操作</h1></center>

---

### 创建可压缩的 tar 包

创建 tar 归档文件与创建 .zip 归档文件非常类似，主要不同点在于我们将所有数据都写入相同的 writer 中，并且在写入文件的数据之前必须写入完整的头部，而非仅仅是一个文件名。

我们在该 pack 程序的实现中使用了 createTar() 和 writeFileToTar() 函数。

```go
func createTar(filename string, files []string) error {
    file, err := os.Create(filename)
    if err != nil {
        return err
    }
    defer file.Close()
    var fileWriter io.WriterCloser = file
    if strings.HasSuffix(filename, ".gz")  {
        fileWriter = gzip.NewWriter(file)
        defer fileWriter.Close()
    }
    writer := tar.NewWriter(fileWriter)
    defer writer.Close()
    for name := range files {
        if err := writeFileToTar(writer, name); err != nil {
            return err
        }
    }
    return nil
}
```

该函数创建了包文件，而且如果扩展名显示该 tar 包需要被压缩则添加一个 gzip 过滤。gzip.NewWriter() 函数返回一个 gzip.Writer 值，它满足 io.WriteCloser 接口（正如打开的 os.File 一样)。

一旦文件准备好写入，我们创建一个 \*tar.Writer 往其中写入数据。然后迭代所有文件并将每一个写入归档文件。

```go
func writeFileToTar(writer *tar.Writer, filename string) error {
    file, err := os.Open(filename)
    if err != nil {
        return err
    }
    defer file.Close()
    stat, err := file.Stat()
    if err != nil {
        return err
    }
    header := &tar.Header{
        Name:      sanitizedName(filename),
        Mode:      int64(stat.Mode()),
        Uid:       os.Getuid(),
        Gid:       os.Getuid(),
        Size:      stat.Size(),
        ModTime:   stat.ModTime(),
    }
    if err = writer.WriteHeader(header); err != nil {
        return err
    }
    err = io.Copy(writer, file)
    return err
}
```

函数首先打开需要处理的文件并设置延迟关闭。然后调用 Stat() 方法取得文件的模式、大小以及修改日期/时间。这些信息用于填充 \*tar.Header，每个文件都必须创建一个 tar.Header 结构并写入到 tar 归档文件里，（此外，我们设置了头部的用户以及组 ID，这会在类 Unix 系统中用到。）我们必须至少设置头部的文件名（其 Name 字段）以及表示文件大小的 Size 字段，否则这个 .tar 包就是非法的。

当 \*tar.Header 结构体创建好后，我们将它写入归档文件，再接着写入文件的内容。

### 解开 tar 归档文件

解开 tar 归档文件比创建 tar 归档文档稍微简单些。然而，跟解开 .zip 文件一样，如果归档文件中的某些文件名包含路径，必须重建目录结构。

```go
func unpackTar(filename string) error {
    file, err := os.Open(filename)
    if err != nil {
        return err
    }
    defer file.Close()
    var fileReader io.ReadCloser = file
    if strings.HasSuffix(filename, ".gz") {
        if fileReader, err = gzip.NewReader(file); err != nil {
            return err
        }
        defer fileReader.Close()
    }
    reader := tar.NewReader(fileReader)
    return unpackTarFiles(reader)
}
```

该方法首先按照 Go 语言的常规方式打开归档文件，并延迟关闭它。如果该文件使用了 gzip 压缩则创建一个 gzip 解压缩过滤器并延迟关闭它。gzip.NewReader() 函数返回一个 gzip.Reader 值，正如打开一个常规文件（类型为 os.File）一样，它也满足 io.ReadCloser 接口。

设置好了文件 reader 之后，我们创建一个 \*tar.Reader 来从中读取数据，并将接下来的工作交给一个辅助函数。

```go
func unpackTarFiles(reader *tar.Reader) error {
    for {
        header, err := reader.Next()
        if err != nil {
            if err == io.EOF {
                return nil // OK
            }
            return err
        }
        filename := sanitizedName(header.Name)
        switch header.Typeflag {
            case tar.TypeDir:
                if err = os.MkdirAll(filename, 0755); err != nil {
                    return err
                }
            case tar.TypeReq:
                if err = unpackTarFile(filename, heaer.Name, reader); err != nil{
                    return err
                }
        }
    }
    return nil
}
```

该函数使用一个无限循环来迭代读取归档文档中的每一项，直到遇到 io.EOF（或者直到遇到错误）。tar.Next() 方法返回压缩文档中第一项或者下一项的 \*tar.Header 结构体，失败则报告错误。如果错误值为 io.EOF，则意味着读取文件结束，因此返回一个空的错误值。

得到 \*tar.Header 后，根据该头部的 Name 字段创建一个净化过的文件名。然后，通过该项的类型标记来进行 switch。对于该简单示例，我们只考虑目录和常规文件，但实际上，归档文件中还可以包含许多其他类型的项（如符号链接）。

如果该项是一个目录，我们按照处理 .zip 文件时所采用的方法创建该目录。而如果该项是一个常规文件，我们将其工作交给另一个辅助函数。

```go
func unpackTarFile(filename, tarFilename string, reader *tar.Reader) error{
    writer, err := os.Create(filename)
    if err != nil {
        return err
    }
    defer writer.Close()
    if err := io.Copy(writer, reader); err != nil {
        return err
    }
    if filename == tarFilename {
        fmt.Printin(filename)
    } else {
        fmt.Printf("％s [%s]\n", filename, tarFilename)
    }
    return nil
}
```

针对归档文件中的每一项，该函数创建了一个新文件，并延迟关闭它。然后，它将归档文件的下一项数据复制到该文件中。同时，正如在 unpackZippedFile() 函数中所做的那样，我们将刚创建文件的文件名打印到终端，如果净化过的文件名与原始文件名不同，则在方括号中给出原始文件名。

至此，我们完成了对压缩和归档文件及常规文件处理的介绍。Go 语言使用 io.Reader、io.ReadCloser、io.Writer 和 io.WriteCloser 等接口处理文件的方式让开发者可以使用相同的编码模式来读写文件或者其他流（如网络流或者甚至是字符串），从而大大降低了难度。
