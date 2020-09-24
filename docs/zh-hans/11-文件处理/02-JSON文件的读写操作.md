<center><h1>JSON文件的读写操作</h1></center>

---

JSON 是一种使用 UTF-8 编码的纯文本格式。由于写起来比 XML 格式方便，并且(通常)更为紧凑，而所需的处理时间也更少，JSON 格式已经越来越流行，特别是在通过网络连接传送数据方面。

这里是一个简单的发票数据的 JSON 表示，但是它省略了该发票的第二项的大部分字段。

```json
{
    "Id": 4461,
    "Customerld": 917,
    "Raised": "2012-07-22",
    "Due": "2012-08-21",
    "Paid": true,
    "Note": "Use trade ent rance",
    "Items":[
        {
            "Id": "AM2574",
            "Price": 415.8,
            "Quantity": 5,
            "Note": ""
        },
        {
            "Id": "MI7296",
            ...
        }
    ]
}
```

通常，encodeing/json 包所写的 JSON 数据没有任何不必要的空格，但是这里我们为了更容易看明白数据的结构而使用了缩进和空白来展示它。虽然 encoding/json 包支持 time.Times，但是我们通过自己实现自定义的 MarshalJSON() 和 UnmarshalJSON() Invoice 方法来处理发票的开具和到期日期。

这样我们就可以存储更短的日期字符串 ( 因为对于我们的数据来说，其时间部分始终为 0)，例如“2012-09-06”，而非整个日期/时间值，如 "2012-09-06T00:00:00Z"。

### 写 JSON 文件

我们创建了一个基于空结构体的类型，它定义了与 JSON 相关的 Marshallnvoices() 和 UnmarshalInvoices() 方法。

```go
type JSONMarshaler struct{}
```

该类型满足我们在前文看到的 InvoicesMarshaler 和 InvoicesUnmarshaler 接口。

这里的方法使用 encoding/json 包中标准的 Go 到 JSON 序列化函数将 []Invoice 项中的所有数据以 JSON 格式写入一个 io.Writer 中。该 writer 可以是 os.Create() 函数返回的 os.File，或者是 gzip.NewWriter() 函数返回的 \*gzip.Writer，或者是任何满足 io.Writer 接口的其他值。

```go
func (JSONMarshaler) Marshallnvoices(writer io.Writer, invoices []*Invoice) error {
    encoder := j son.NewEncoder(writer)
    if err := encoder.Encode(fileType); err != nil {
        return err
    }
    if err := encoder.Encode(fileVersion); err != nil {
        return err
    }
    return encoder.Encode(invoices)
}
```

JSONMarshaler 类型没有数据，因此我们没必要将其值赋值给一个接收器变量。

函数开始处，我们创建了一个 JSON 编码器，它包装了 io.Writer，可以接收我们写入的支持 JSON 编码的数据。

我们使用 json.Encoder.Encode() 方法来写入数据。该方法能够完美地处理发票切片，其中每个发票都包含一到多个项的切片。该方法返回一个错误值或者空值 nil。如果返回的是一个错误值，则立即返回给调用者。

文件的类型和版本不是必须写入的，但在后面一个练习中会看到，这样做是为了以后更容易更改文件格式 (例如，为了适应 Invoice 和 Item 结构体中额外的字段 )，以及为了能够同时支持读取新旧格式的数据。

需注意的是，该方法实际上与它所编码的数据类型无关，因此很容易创建类似函数用于写入其他可 JSON 编码的数据。另外，只要新的文件格式中新增的字段是导出的且支持 JSON 编码，该 JSONMarshaler.Marshallnvoices() 方法无需做任何更改。

如果这里所给出的代码就是 JSON 相关代码的全部，这样当然可以很好地工作。然而，由于我们希望更好地控制 JSON 的输出，特别是对 time.Time 值的格式化，我们还为 Invoice 类型提供了一个满足 json.Marshaler 接口的 MarshalJSON() 方法。

json.Encode() 函数足够智能，它会去检查所需编码的值是否支持 json.Marshaler 接口，如果支持，该函数会使用该值的 MarshalJSON() 方法而非内置的编码代码。

```go
type JSONInvoice struct {
    Id         int
    CustomerId int
    Raised     st ring      // Invoice 结构体中的 time.Time
    Due        string       // Invoice 结构体中的 time.Time
    Paid       bool
    Note       string
    Items      []*Item
}
func (invoice Invoice) MarshalJSON()([]byte, error) {
    jsonlnvoice := JSONInvoice {
        invoice.Id,
        invoice.Customerld,
        invoice.Raised.Format(dateFormat),
        invoice.Due.Format(dateFormat),
        invoice.Paid,
        invoice.Note,
        invoice.Items,
    }
    return json.Marshal(jsonlnvoice)
}
```

该自定义的 Invoice.MarshalJSON() 方法接受一个已有的 Invoice 值，返回一个该数据 JSON 编码后的版本。该函数的第一个语句简单地将发票的各个字段复制到自定义的 JSONInvoice 结构体中，同时将两个 time.Time 值转换成字符串。

由于 JSONInvoice 结构体的字段都是布尔类型、数字或者字符串，该结构体可以使用 json.Marshal() 函数进行编码，因此我们使用该函数来完成工作。

为了将日期/时间 ( 即 time.Time 值 ) 以字符串的形式写入，我们必须使用 time.Time.Format() 方法。该方法接受一个格式字符串，它表示该日期/时间值应该如何写入。

该格式字符串非常特殊，必须是一个 Unix 时间 1 136 243 045 的字符串表示，即精确的日期/时间值 2006-01-02T15:04:05Z07:00，或者跟这里一样，使用该日期/时间值的子集。该特殊的日期/时间值是任意的，但必须是确定的，因为没有其他的值来声明日期、时间以及日期/时间的格式。

如果我们想自定义日期/时间格式，它们必须按照 Go 语言的日期/时间格式来写。假如我们要以星期、月、日和年的形式来写日期，我们必须使用“Mon, Jan 02, 2006”这种格式，或者如果我们希望删除前导的 0，就必须使用"Mon, Jan \_2, 2006”这种格式。time 包的文档中有完整的描述，并列出了一些预定义的格式字符串。

### 读 JSON 文件

读 JSON 数据与写 JSON 数据一样简单，特别是当将数据读回与写数据时类型一样的变量时。JSONMarshaler.UnMarshallnvoices() 方法接受一个 io.Reader 值，该值可以是一个 os.Open() 函数返回的 os.File 值，或者是一个 gzip.NewReader() 函数返回的 gzip.Reader 值，也可以是任何满足 io.Reader 接口的值。

```go
func (JSONMarshaler) Unmarshallnvoices(reader io.Reader) ([]*Invoice, error){
    decoder := json.NewDecoder(reader)
    var kind string
    if err := decoder.Decode(&king); err != nil {
        return nil, err
    }
    if kind != fileType {
        return nil, errors.New ("Cannot read non-invoices json file")
    }
    var version int
    if err := decoder.Decode(&version); err != nil {
        return nil, err
    }
    if version > fileVersion {
        return nil, fmt.Error("version %d is too new to read", version)
    }
    var invoices []*Invoice
    err := decoder.Decode(&invoices)
    return invoices, err
}
```

我们需读入 3 项数据：文件类型、文件版本以及完整的发票数据。json.Decoder.Decode() 方法接受一个指针，该指针所指向的值用于存储解码后的 JSON 数据，解码后返回一个错误值或者 nil。

我们使用前两个变量 (kind 和 version) 来保证接受一个 JSON 格式的发票文件，并且该文件的版本是我们能够处理的。然后程序读取发票数据，在该过程中，随着 json.Decoder.Decode() 方法所读取发票数的增多，它会增加 invoices 切片的长度，并将相应发票的指针 ( 及其项目 ) 保存在切片中，这些指针是 Unmarshallnvoices 函数在必要时实时创建的。最后，该方法返回解码后的发票数据和一个 nil 值。或者，如果解码过程中遇到了问题则返回一个 nil 值和一个错误值。

如果我们之前纯粹依赖于 json 包内置的功能把数据的创建及到期日期按照默认的方式序列化，那么这里给出的代码已经足以反序列化一个 JSON 格式的发票文件。然而，由于我们使用自定义的方式来序列化数据的建立和到期日期 time.Times (只存储日期部分 )，我们必须提供一个自定义的反序列化方法，该方法理解我们的自定义序列化流程。

```go
func (invoice *Invoice) UnmarshalJSON(data []byte) (err error) {
    var jsonlnvoice JSONInvoice
    if err = json.Unmarshal(data, &jsonlnvoice); err != nil {
        return err
    }
    var raised, due time.Time
    if raised, err = time.Parse(dateFormat, jsonlnvoice.Raised);
        err != nil {
            return err
        }
    if due, err = time.Parse(dateFormat, jsonlnvoice.Due); err != nil {
        return err
    }
    *invoice = Invoice {
        jsonlnvoice.Id,
        jsonlnvoice.Customerld,
        raised,
        due,
        jsonlnvoice.Paid,
        jsonlnvoice.Note,
        jsonlnvoice.Items,
    }
    return nil
}
```

该方法使用与前面一样的 JSONInvoice 结构体，并且依赖于 json.Unmarshal() 函数来填充数据。然后，我们将反序列化后的数据以及转换成 time.Time 的日期值赋给新创建的 Invoice 变量。

json.Decoder.Decode() 足够智能会检查它需要解码的值是否满足 json.Unmarshaler 接口，如果满足则使用该值自己的 UnmarshalJSON() 方法。

如果发票数据因为新添加了导出字段而发生改变，该方法能继续正常工作的前提是我们必须让 Invoice.UnmarshalJSON() 方法也能处理版本变化。另外，如果新添加字段的零值不可被接受，那么当以原始格式读文件的时候，我们必须对数据做一些后期处理，并给它们一个合理的值。

虽然要支持两个或者更多个版本的 JSON 文件格式有点麻烦，但 JSON 是一种很容易处理的格式，特别是如果我们创建的结构体的导出字段比较合理时。同时，json.Encoder.Encode() 函数和 json.Decoder.Decode() 函数也不是完美可逆的，这意味着序列化后得到的数据经过反序列化后不一定能够得到原始的数据。因此，我们必须小心检查，保证它们对我们的数据有效。

顺便提一下，还有一种叫做 BSON (Binary JSON) 的格式与 JSON 非常类似，它比 JSON 更为紧凑，并且读写速度也更快。godashboard.appspot.com/project 网页上有一个支持 BSON 格式的第三方包 (gobson)。
