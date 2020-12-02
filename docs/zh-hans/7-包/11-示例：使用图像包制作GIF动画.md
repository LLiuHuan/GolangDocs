<center><h1>示例：使用图像包制作GIF动画</h1></center>

---

通过示例来介绍一下 Go 语言标准的图像包的使用，用来创建一系列的位图图像，然后将位图序列编码为 GIF 动画。下面的图像叫作利萨茹图形，是 20 世纪 60 年代科幻片中的纤维状视觉效果。利萨茹图形是参数化的二维谐振曲线，如示波器 x 轴和 y 轴馈电输入的两个正弦波。

<div align=center> 
    <img src="img/7-包/11-示例：使用图像包制作GIF动画/四种利萨茹图形.gif"/> 
    <p>图：四种利萨茹图形</p>
</div>

这段代码里有几个新的组成，包括 const 声明、结构体以及复合字面量。不像大多数例子，本例还引入了浮点运算。这个示例的主要目的是提供一些思路，表明 Go 语言看起来是怎样的，以及利用 Go 语言和它的库可以轻易完成哪些事情，这里只简短地讨论这几个主题。

```go
// lissajous 产生随机利萨茹图形的 GIF 动画
package main
import (
    "image"
    "image/color"
    "image/gif"
    "io"
    "math"
    "math/rand"
    "os"
)
var palette = []color.Color{color.White, color.Black}
const (
    whiteIndex = 0 //画板中的第一种颜色
    blackIndex = 1 //画板中的下一种颜色
)
func main() {
    rand.Seed(time.Now().UTC().UnixNano())
    if len(os.Args) > 1 && os.Args[1] == "web" {
        handler := func(w http.ResponseWriter, r *http.Request) {
            lissajous(w)
        }
        http.HandleFunc("/", handler)
        log.Fatal(http.ListenAndServe("localhost:8000", nil))
        return
    }
    lissajous(os.Stdout)
}
func lissajous(out io.Writer) {
    const (
        cycles = 5      //完整的x振荡器变化的个数
        res = 0.001     //角度分辨率
        size = 100      //图像画布包含[-size. .+size]
        nframes = 64    //动画中的帧数
        delay = 8       //以10ms为单位的帧间延迟
    )
    freq := rand.Float64() * 3.0 //y 振荡器的相对频率
    anim := gif.GIF{LoopCount: nframes}
    phase := 0.0 // phase differenee
    for i := 0; i < nframes; i++ {
        rect := image.Rect(0, 0, 2*size+1, 2*size+1)
        img := image.NewPaletted(rect, palette)
        for t := 0.0; t < cycles*2*math.Pi; t += res {
            x := math.Sin(t)
            y := math.Sin(t*freq + phase)
            img.SetColorIndex(size+int(x*size+0.5), size+int(y*size+0.5), blackindex)
        }
        phase += 0.1
        anim.Delay = append(anim.Delay, delay)
        anim.Image = append(anim.Image, img)
    }
    gif .EncodeAll(out, &anim) //注意：忽略编码错误
}
```

在导入那些由多段路径如 image/color 组成的包之后，使用路径最后的一段来引用这个包。所以变量 color.White 属于 image/color 包，gif.GIF 属于 image/gif 包。

const 声明用来给常量命名，常量是其值在编译期间固定的量，例如周期、帧数和延迟等数值参数。与 var 声明类似，const 声明可以岀现在包级别（所以这些常量名字在包生命周期内都是可见的）或在一个函数内（所以名字仅在函数体内可见）。常量必须是数字、字符串或布尔值。

表达式[]color.Color{...}和 gif.GIF{...}是复合字面量，即用一系列元素的值初始化 Go 的复合类型的紧凑表达方式。这里，第一个是 slice，第二个是结构体。

gif.GIF 是一个结构体类型。结构体由一组称为字段的值组成，字段通常有不同的数据类型，它们一起组成单个对象，作为一个单位被对待。anim 变量是 gif.GIF 结构体类型。这个结构体字面量创建一个结构体 LoopGount，其值设置为 nframes；其他字段的值是对应类型的零值。结构体的每个字段可以通过点记法来访问，在最后两个赋值语句中，显式更新 anim 结构体的 Delay 和 Image 字段。

lissajous 函数有两个嵌套的循环。外层有 64 个迭代，每个迭代产生一个动画帧。它创建一个 201x201 大小的画板，使用黑和白两种颜色。所有的像素值默认设置为 0（画板中的初始化颜色），这里设置为白色。每一个内层循环通过设置一些像素为黑色产生一个新的图像。

结果使用内置的 append 参数将其追加到 anim 的帧列表中，并且指定 80ms 的延迟。最后帧和延迟的序列被编码成 GIF 格式，然后写入输出流 out。out 的类型是 io.Writer，它可以帮我们输出到很多地方，稍后即可看到。

外层循环运行两个振荡器。x 方向的振荡器是正弦函数，y 方向也是正弦化的，但是它的频率相对于 x 的振动周期是 0〜3 之间的一个随机数，它的相位相对于 x 的初始值为 0，然后随着每个动画帧增加。该循环在 x 振荡器完成 5 个完整周期后停止。每一步它都调用 SetColorIndex 将对应画板上面的 (x,y) 位置涂为黑色，在画板上的值为 1。

main 函数调用 lissajous 函数，直接写到标准输出，所以这个命令产生一个像上面图中所示的那 样的 GIF 动画：

```
$ go build gopl.io/chl/lissajous
$ ./lissajous > out.gif
```
