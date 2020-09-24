<center><h1>XML文件的读写操作</h1></center>

---

XML(extensible Markup Language) 格式被广泛用作一种数据交换格式，并且自成一种文件格式。与 JSON 相比，XML 复杂得多，手动写起来也啰嗦而且乏味得多。

encoding/xml 包可以用在结构体和 XML 格式之间进行编解码,其方式跟 encoding/json 包类似。然而，与 encoding/json 包相比，XML 的编码和解码在功能上更苛刻得多。这部分是由于 encoding/xml 包要求结构体的字段包含格式合理的标签（然而 JSON 格式却不需要）。

同时，Go 1 的 encoding/xml 包没有 xml.Marshaler 接口，因此与编解码 JSON 格式和 Go 语言的二进制格式相比，我们处理 XML 格式时必须写更多的代码。(该问题有望在 Go 1.x 发行版中得以解决。)

这里有个简单的 XML 格式的发票文件。为了适应页面的宽度和容易阅读，我们添加了换行和额外的空白。

```xml
<INVOICE Id="2640" CustomerId="968" Raised="2012-08-27" DUe="2012-09-26" Paid="false"><NOTE>See special Terms &amp; Conditions</NOTE>
    <ITEM Id="MI2419" Price="342.80" Quantity="1"><NOTE></NOTE></ITEM>
    <ITEM Id="OU5941" Price="448.99" Quantity="3"><N0TE>
        &quot;Blue&quot; ordered but will accept &quot;Navy&quot;</NOTE> </ITEM>
    <ITEM Id="IF9284" Price="475.01" Quantity="1"><NOTE></NOTE></ITEM>
    <ITEM Id="TI4394" Price="417.79" Quantity="2"><N0TE></N0TE></ITEM>
    <ITEM Id="VG4325" Price="80.67" Quantity= "5"><NOTE></NOTE></ITEM>
</INVOICE>
```

对于 xml 包中的编码器和解码器而言，标签中如果包含原始字符数据 ( 如 invoice 和 item 中的 Note 字段 ) 处理起来比较麻烦，因此 invoicedata 示例使用了显式的 < NOTE> 标签。

### 写 XML 文件

encoidng/xml 包要求我们使用的结构体中的字段包含 encoding/xml 包中所声明的标签，所以我们不能直接将 Invoice 和 Item 结构体用于 XML 序列化。 因此，我们创建了针对 XML 格式的 XMLInvoices、XMLInvoice 和 XMLItem 结构体来解决这个问题。同时，由于 invoicedata 程序要求我们有并行的结构体集合，因此必须提供一种方式来让它们相互转换。

下面是保存整个数据集合的 XMLInvoices 结构体。

```go
type XMLInvoices struct {
    XMLName xml.Name         'xml:"INVOICES"'
    Version int              'xml:"version, attr"'
    Invoice []*XMLInvoice    'xml:"INVOICE"'
}
```

在 Go 语言中，结构体的标签本质上没有任何语义，它们只是可以使用 Go 语言的反射接口获得的字符串。然而，encoding/xml 包要求我们使用该标签来提供如何将结构体的字段映射到 XML 的信息。

xml.Name 字段用于为 XML 中的标签命名，该标签包含了该字段所在的结构体。以 'xml:",attr"' 标记的字段将成为该标签的属性，字段名字将成为属性名。我们也可以根据自己的喜好使用另一个名字，只需在所给的名字签名加上一个逗号。

这里，我们把 Version 字段当做一个叫做 version 的属性，而非默认的名字 Version。如果标签只包含一个名字，则该名字用于表示嵌套的标签，如此例中的 ＜ TNVOTCE ＞ 标签。有一个非常重要的细节需注意的是，我们把 XMLInvoices 的发票字段命名为 Invoice，而非 Invoices，这是为了匹配 XML 格式中的标签名（不区分大小写）。

下面是原始的 Invoice 结构体，以及与 XML 格式相对应的 XMLInvoice 结构体。

```go
type Invoice struct {
    Id           int
    Customerld   int
    Raised       time.Time
    Due          time.Time
    Paid         bool
    Note         string
    Items        []*1tem
}
type XMLInvoice struct {
    XMLName     xml.Name   'xml: "INVOICE"'
    Id          int        'xml:",attr"'
    CustomerId  int        'xml:",attr"'
    Raised      string     'xml:",attr"'
    Due         string     'xml:",attr"'
    Paid        bool       'xml:",attr"'
    Note        string     'xml:"NOTE"'
    Item        []*XMLItem 'xml:"ITEM"'
}
```

在这里，我们为属性提供了默认的名字。例如，字段 Customerld 在 XML 中对应一个属性，其名字与该字段的名字完全一样。这里有两个可嵌套的标签：＜ NOTE ＞和＜ ITEM ＞，并且如 XMLInvoices 结构体一样，我们把 XMLInvoice 的 item 字段定义成 Item（大小写不敏感）而非 Items，以匹配标签名。

由于我们希望自己处理创建和到期日期（只存储日期），而非让 encoding/xml 包来保存完整的日期/时间字符串，我们为它们在 XMLInvoice 结构体中定义了相应的 Raised 和 Due 字段。

下面是原始的 Item 结构体，以及与 XML 相对应的 XMLItem 结构体。

```go
type Item struct {
    Id        string
    Price     float64
    Quantity  int
    Note      string
}
type XMLItem struct {
    XMLName  xml.Name     'xml:"ITEM"'
    Id       string       'xml:",attr"'
    Price    float64      'xml:",attr"'
    Quantity int          'xml:",attr"'
    Note     string       'xml:"NOTE"'
}
```

除了作为嵌套的 ＜ NOTE ＞ 标签的 Note 字段和用于保存该 XML 标签名的 XMLName 字段之外，XMLItem 的字段都被打上了标签以作为属性。

正如处理 JSON 格式时所做的那样，对于 XML 格式，我们创建了一个空的结构体并关联了 XML 相关的 MarshalInvoices() 方法和 UnmarshalInvoices() 方法。

```go
type XMLMarshaler struct{}
```

该类型满足前文所述的 InvoicesMarshaler 和 InvoiceUnmarshaler 接口。

```go
func (XMLMarshaler) Marshallnvoices(writer io.Writer, invoices []*Invoice) error {
    if err := writer.Writer([]byte(xml.Header)); err != nil {
        return err
    }
    xmlinvoices := XMLInvoicesForInvoices(invoices)
    encoder := xml.NewEncoder(writer)
    return encoder.Encode(xmlinvoices)
}
```

该方法接受一个 io.Writer (也就是说，任何满足 io.Writer 接口的值如打开的文件或者打开的压缩文件)，以用于写入 XML 数据。该方法从写入标准的 XML 头部开始 ( 该 xml.Header 常量的末尾包含一个新行)。

然后，它将所有的发票数据及其项写入相应的 XML 结构体中。这样做虽然看起来会耗费与原始数据相同的内存，但是由于 Go 语言的字符串是不可变的，因此在底层只将原始数据字符串的引用复制到 XML 结构体中，因此其代价并不是我们所看到的那么大。而对于直接使用带有 XML 标签的结构体的应用而言，其数据没必要再次转换。

一旦填充好 xmlInvoices( 其类型为 XMLInvoices) 后，我们创建了一个新的 xml.Encoder，并将我们希望写入数据的 io.Writer 传给它。然后，我们将数据编码成 XML 格式，并返回编码器的返回值，该值可能为一个 error 值也可能为 nil。

```go
func XMLInvoicesForlnvoices(invoices []*Invoice) *XMLInvoices {
    xmlinvoices :=• &XMLInvoices{
        Version: fileVersion,
        Invoice: make([]*XMLInvoice, 0, len(invoices)),
    }
    for _, invoice := range invoices {
        xmlInvoices.Invoice = append(xmlinvoices.Invoice,
            XMLInvoiceForInvoice (invoice))
    }
    return xmlInvoices
}
```

该函数接受一个 []Invoice 值并返回一个 XMLInvoices 值，其中包含转换成 XMLInvoices(还包含 XMLItems 而非 \*Items) 的所有数据。该函数又依赖于 XmlInvoiceForInvoice() 函数来为其完成所有工作。

我们不必手动填充 xml.Name 字段（除非我们想使用名字空间），因此在这里，当创建 \*XMLInvoices 的时候，我们只需填充 Version 字段以保证我们的标签有一个 version 属性，例如 ＜ INVILES verion="100"＞。

同时，我们将 Invoice 字段设置成一个空间足够容纳所有的发票数据的空切片。这样做不是严格必须的，但是与将该字段的初始值留空相比，这样做可能更高效，因为这样做意味着调用内置的 append() 函数时无需分配内存和复制数据以扩充切片容量。

```go
func XMLInvoiceForlnvoice(invoice *Invoice) *XMLInvoice {
    xmlInvoice := &XMLInvoice{
        Id:           invoice.id,
        CustomerId:   invoice.CustomerId,
        Raised:       invoice.Raised.Format(dateFormat),
        Due:          invoice.Due.Format(dateFormat),
        Paid:         invoice.Paid,
        Note:         invoice.Note,
        Item:         make([]*XMLItem, 0, len(invoice.Items)),
    }
    for _, item := range invoice.Items {
        xmlItem := &XMLItem {
            Id:         item.Id,
            Price:      item.Price,
            Quantity:   item.Quantity,
            Note:       item.Note,
        }
        xmlInvoice.Item = append(xmlInvoice.Item, xmlItem)
    }
    return xmlInvoice
}
```

该函数接受一个 Invoice 值并返回一个等价的 XMLInvoice 值。该转换非常直接，只需简单地将 Invoice 中每个字段的值复制至 XMLInvoice 字段中。由于我们选择自己来处理创建以及到期日期（因此我们只需存储日期而非完整的日期/时间），我们只需将其转换成字符串。

而对于 Invoice.Items 字段，我们将每一项转换成 XMLItem 后添加到 XMLInvoice.Item 切片中。与前面一样，我们使用相同的优化方式，创建 Item 切片时分配了足够多的空间以避免 append() 时需要分配内存和复制数据。前文阐述 JSON 格式时我们已讨论过 time.Time 值的写入。

最后需要注意的是，我们的代码中没有做任何 XML 转义，它是由 xml.Encoder.Encode() 方法自动完成的。

### 读 XML 文件

读 XML 文件比写 XML 文件稍微复杂，特别是在必须处理一些我们自定义的字段的时候（例如日期）。但是，如果我们使用合理的打上 XML 标签的结构体，就不会复杂。

```go
func (XMLMarshaler) UnmarshalInvoices(reader io.Reader)([]*Invoice, error) {
    xmlInvoices := &XMLInvoices{}
    decoder := xml.NewDecoder(reader)
    if err := decoder.Decode(xmlInvoices); err != nil {
        return nil, err
    }
    if xmlInvoices.Version > fileVersion {
        return nil, fmt.Errorf ("version %d is too new to read", xmlInvoices.Version)
    }
    return xmlInvoices.Invoices()
}
```

该方法接受一个 io.Reader( 也就是说，任何满足 io.Reader 接口的值如打开的文件或者打开的压缩文件)，并从其中读取 XML。该方法的开始处创建了一个指向空 XMLInvoices 结构体的指针，以及一个 xml.Decoder 用于读取 io.Reader。

然后，整个 XML 文件由 xml.Decoder.Decode() 方法解析，如果解析成功则将 XML 文件的数据填充到该 \*XMLInvoices 结构体中。如果解析失败（例如，XML 文件语法有误，或者该文件不是一个合法的发票文件），那么解码器会立即返回错误值给调用者。

如果解析成功，我们再检查其版本，如果该版本是我们能够处理的，就将该 XML 结构体转换成我们程序内部使用的结构体。当然，如果我们直接使用带 XML 标签的结构体，该转换步骤就没必要了。

```go
func (xmllnvoices *XMLInvoices) Invoices() (invoices []*Invoice, err error){
    invoices = make([]*Invoice, 0, len(xmllnvoices.Invoice))
    for _, XMLInvoice := range xmlInvoices.Invoice {
        invoice, err := xmlInvoice.Invoice()
        if err != nil {
            return nil, err
        }
        invoices = append(invoices, invoice)
    }
    return invoices, nil
}
```

该 XMLInvoices.Invoices() 方法将一个 XMLInvoices 值转换成一个 []Invoice 值，它是 XmllnvoicesForInvoices() 函数的逆反操作，并将具体的转换工作交给 XMLInvoice.Invoice() 方法完成。

```go
func (xmlInvoice *XMLInvoice) Invoice() (invoice *Invoice, err error) {
    invoice = &Invoice{
        Id:          xmlInvoice.Id,
        CustomerId:  xmlInvoice.CustomerId,
        Paid:        xmlInvoice.Paid,
        Note:        strings.TrimSpace(xmlInvoice.Note),
        ems:         make([]*Item, 0, len(xmlInvoice.Item)),
    }
    if invoice.Raised, err = time.Parse(dateFormat, xmlInvoice.Raised);
        err != nil {
        return nil, err
    }
    if invoice.Due, err = time.Parse(dateFormat, xmlInvoice.Due);
        err != nil{
        return nil, err
    }
    for _, xmlItem := range xmlInvoice.Item {
        item := &Item {
            Id:         xmlItem.Id,
            Price:      xmlItem.Price,
            Quantity:   xmlItem.Quantity,
            Note:       strings.TrimSpace(xmlItem.Note),
        }
        invoice.Items = append(invoice.Items, item)
    }
    return invoice, nil
}
```

该方法用于返回与调用它的 XMLInvoice 值相应的 Invoice 值。

该方法在开始处创建了一个 Invoice 值，其大部分字段都由来自 XMLInvoice 的数据填充，而 Items 字段则设置成一个容量足够大的空切片。

然后，由于我们选择自己处理这些，因此手动填充两个日期/时间字段。time.Parse() 函数接受一个日期/时间格式的字符串（如前所述，该字符串必须基于精确的日期/时间值，如 2006-01-02T15:04:05Z07:00），以及一个需要解析的字符串，并返回等价的 time.Time 值和 nil，或者，返回一个 nil 和一个错误值。

接下来是填充发票的 Items 字段，这是通过迭代 XMLInvoice 的 Item 字段中的 XMLItems 并创建相应的 Items 来完成的。最后，返回 \*Invoice。

正如写 XML 时一样，我们无需关心对所读取的 XML 数据进行转义，xml.Decoder.Decode() 函数会自动处理这些。

```
xml 包支持比我们这里所需的更为复杂的标签，包括嵌套。例如，标签名为 'xml:"Books>Author"' 产生的是 < Books>< Author>content< /Author>< /Books> 这样的 XML 内容。同时，除了 'xml:", attr"' 之外，该包还支持 'xml:",chardata"' 这样的标签表示将该字段当做字符数据来写，支持 'xml:",innerxml"' 这样的标签表示按照字面量来写该字段，以及 'xml:",comment"' 这样的标签表示将该字段当做 XML 注释。因此，通过使用标签化的结构体，可以充分利用好这些方便的编码解码函数，同时合理控制如何读写 XML 数据。
```
