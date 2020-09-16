<center><h1>Test功能测试函数多维数组</h1></center>

---

每一个测试文件必须导入 testing 包。这些函数的函数签名如下所示：

```go
func TestName(t *testing.T) {
    //...
}
```

功能测试函数必须以 Test 开头，可选的后缀名称必须以大写字母开头：

```go
func TestSin(t *testing.T) {/*...*/}
func TestCos(t *testing.T) {/*...*/}
func TestLog(t *testing.T) {/*...*/}
```

参数 t 提供了汇报测试失败和日志记录的功能。定义一个示例包，代码如下所示，这个包包含一个函数 IsPalindrome，用来判断一个字符串是否是回文字符串（这个函数在字符串是回文字符串的情况下对于每个字节检查了两次）。

```go
// 包 word 提供了文字游戏相关的工具函数
package word
// IsPalindrome 判断一个字符串是否是回问字符串
// (Our first attempt.)
func IsPalindrome(s string) bool {
    for i := range s {
        if s[i] != s[len(s)-1-i] {
            return false
        }
    }
    return true
}
```

然后再定义一个包，代码如下，包中含有两个功能测试函数 TestPalindrome 和 TestNonPalindrome。两个函数都检查 isPalindrome 是否针对单个输入参数给出了正确的结果，并且用 t.Error 来报错。

```go
package word
import "testing"
func TestPalindrome(t *testing.T) {
    if !IsPalindrome("detartrated") {
        t.Error(`IsPalindrome("detartrated") = false`)
    }
    if !IsPalindrome("kayak") {
        t.Error(`IsPalindrome("kayak") = false`)
    }
}
func TestNonPalindrome(t *testing.T) {
    if IsPalindrome("palindrome") {
        t.Error(`IsPalindrome("palindrome") = true`)
    }
}
```

go test（或者 go build）命令在不指定包参数的情况下，以当前目录所在的包为参数。可以用下面的命令来编译和运行测试：

```
$ cd $GOPATH/src/gopl.io/chll/word1
$ go test
ok gopl.io/chll/word1 0.008s
```

测试通过，但是没过多久就开始有 bug 提交过来了。有个用户抱怨说 IsPalindrome 函数无法识别“été”。还另一个用户发现程序无法判断出"A man, a plan, a canal: Panama"。 这些特定的小 bug 自然导致了新的测试用例的产生。

```go
func TestFrenchPalindrome(t *testing.T) {
    if !IsPalindrome("été") {
        t.Error(`IsPalindrome("été") = false`)
    }
}
func TestCanalPalindrome(t *testing.T) {
    input := "A man, a plan, a canal: Panama"
    if !IsPalindrome(input) {
        t.Errorf(`IsPalindrome(%q) = false`, input)
    }
}
```

由于 input 很长，为了避免写两次，这里使用了函数 Errorf，这个函数和 Printf 一样提供了格式化功能。

当添加这两个新的测试之后，go test 命令失败了，给出如下错误消息：

```
$ go test
--- FAIL: TestFrenchPalindrome (0.00s)
     word_test.go:32: IsPalindrome("été") = false
--- FAIL: TestCanalPalindrome (0.00s)
    word_test.go:39: IsPalindrome("A man, a plan, a canal: Panama") = false
FAIL
FAIL    gopl.io/chll/word1    5.263s
```

比较好的实践是先写测试然后发现它触发的错误和用户 bug 报告里面的一致。只有这个时候，我们才能确信我们修复的内容是针对这个出现的问题的。

另外，运行 go test 比手动测试 bug 报告中的内容要快得多，测试可以让我们顺序地检查内容。如果一个测试套件 (test suite) 里面有很多测试用例，我们可以选择性地测试用例来加加测试过程。

选项 -v 可以输出包中每个测试用例的名称和执行的时间：

```
$ go test -v
=== RUN   TestPalindrome
---    PASS: TestPalindrome (0.00s)
=== RUN   TestNonPalindrome
---    PASS: TestNonPalindrome (0.00s)
=== RUN   TestFrenchPalindrome
---    FAIL: TestFrenchPalindrome (0.00s)
        word_test.go:32: IsPalindrome("été") = false
=== RUN   TestCanalPalindrome
---    FAIL: TestCanalPalindrome (0.00s)
        word_test.go:39: IsPalindrome("A man, a plan, a canal: Panama") = false
FAIL
exit status 1
FAIL    gopl.io/chll/word1    5.213s
```

选项 -run 的参数是一个正则表达式，它可以使得 go test 只运行那些测试函数名称匹配给定模式的函数：

```
$ go test -v -run="French|Canal"
=== RUN   TestFrenchPalindrome
---    FAIL:   TestFrenchPalindrome (0.00s)
        word_test.go:32: IsPalindrome("été") = false
=== RUN   TestCanalPalindrome
---    FAIL:   TestCanalPalindrome (0.00s)
        word_test.go:39: IsPalindrome("A man, a plan, a canal: Panama") = false
FAIL
exit status 1
FAIL    gopl.io/chll/word1    5.189s
```

当然，一旦我们使得选择的测试用例通过之后，在我们提交更改之前，我们必须重新使用不带开关的 go test 来运行一次整个测试套件。

现在的任务是修复 bug。经过迅速地调查发现第一个 bug 的原因是函数 IsPalindrome 使用字节序列而不是字符序列来比较，因此那些非 ASCII 字符（例如"été"中的"é"）就使得程序困惑了。第二个 bug 的原因是没有忽略空格、标点符号和字母大小写。

有了教训后，我们仔细地重写了这个函数：

```go
//包 word 提供了文字游戏相关的工具函数
package word
import "unicode"
// IsPalindrome 判断一个字符串是否是回文字符串
// 忽略字母大小写，以及非字母字符
func IsPalindrome(s string) bool {
    var letters []rune
    for _, r := range s {
        if unicode.IsLetter(r) {
            letters = append(letters, unicode.ToLower(r))
        }
    }
    for i := range letters {
        if letters[i] != letters[len(letters)-1-i] {
            return false
        }
    }
    return true
}
```

我们还写了更加全面的测试用例，把前面的用例和新的用例结合到一个表里面。

```go
package word
import (
    "fmt"
    "math/rand"
    "time"
)
import "testing"
func TestIsPalindrome(t *testing.T) {
    var tests = []struct {
        input string
        want  bool
    }{
        {"", true},
        {"a", true},
        {"aa", true},
        {"ab", false},
        {"kayak", true},
        {"detartrated", true},
        {"A man, a plan, a canal: Panama", true},
        {"Evil I did dwell; lewd did I live.", true},
        {"Able was I ere I saw Elba", true},
        {"été", true},
        {"Et se resservir, ivresse reste.", true},
        {"palindrome", false}, // 非回文
        {"desserts", false},   // 半回文
    }
    for _, test := range tests {
        if got := IsPalindrome(test.input); got != test.want {
            t.Errorf("IsPalindrome(%q) = %v", test.input, got)
        }
    }
}
func BenchmarkIsPalindrome(b *testing.B) {
    for i := 0; i < b.N; i++ {
        IsPalindrome("A man, a plan, a canal: Panama")
    }
}
func ExampleIsPalindrome() {
    fmt.Println(IsPalindrome("A man, a plan, a canal: Panama"))
    fmt.Println(IsPalindrome("palindrome"))
    // Output:
    // true
    // false
}
//randompalindrome返回一个回文，回文的长度和内容
//来自伪随机数生成器rng。
func randomPalindrome(rng *rand.Rand) string {
    n := rng.Intn(25) // 随机长度不超过24
    runes := make([]rune, n)
    for i := 0; i < (n+1)/2; i++ {
        r := rune(rng.Intn(0x1000)) // '\u0999'之前的随机运行
        runes[i] = r
        runes[n-1-i] = r
    }
    return string(runes)
}
func TestRandomPalindromes(t *testing.T) {
    // 初始化伪随机数生成器
    seed := time.Now().UTC().UnixNano()
    t.Logf("Random seed: %d", seed)
    rng := rand.New(rand.NewSource(seed))
    for i := 0; i < 1000; i++ {
        p := randomPalindrome(rng)
        if !IsPalindrome(p) {
            t.Errorf("IsPalindrome(%q) = false", p)
        }
    }
}
```

新的测试可以通过了：

```
$ go test gopl.io/chll/word2
ok    gopl.io/chll/word2    0.015s
```

这种基于表的测试方式在 Go 里面很常见。根据需要添加新的表项目很直观，并且由于断言逻辑没有重复，因此我们可以花点精力让输出的错误消息更好看一点。

当前调用 t.Errorf 输出的失败的测试用例信息没有包含整个跟踪栈信息，也不会导致程序宕机或者终止执行，这和很多其他语言的测试框架中的断言不同。测试用例彼此是独立的。如果测试表中的一个条目造成测试失败，那么其他的条目仍然会继续测试，这样我们就可以在一次测试过程中发现多个失败的情况。

如果我们真的需要终止一个测试函数，比如由于初始化代码失败或者避免已有的错误产生令人困惑的输出，我们可以使用 t.Fatal 或者 t.Fatalf 函数来终止测试。这些函数的调用必须和 Test 函数在同一个 goroutine 中，而不是在测试创建的其他 goroutine 中。

测试错误消息一般格式是 "f(x)=y, wantz"，这里 f(x) 表示需要执行的操作和它的输入，y 是实际的输出结果，z 是期望得到的结果。出于方便，对于 f(x) 我们会使用 Go 的语法，比如在上面回文的例子中，我们使用 Go 的格式化来显示较长的输入，避免重复输入。

在基于表的测试中，输出 x 是很重要的，因为一条断言语句会在不同的输入情况下执行多次。错误消息要避免样板文字和冗余信息。在测试一个布尔函数的时候，比如上面的 IsPalindrome，省略 "want z" 部分，因为它没有给出有用信息。如果 x、y、z 都比较长，可以输出准确代表各部分的概要信息。
