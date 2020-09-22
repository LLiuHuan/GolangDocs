<center><h1>示例：实现有限状态机（FSM）</h1></center>

---

有限状态机（Finite-State Machine, FSM），表示有限个状态及在这些状态间的转移和动作等行为的数学模型。本节将通过示例来为大家演示如何实现状态接口、状态管理器及一系列的状态和使用状态的逻辑。

### 状态的概念

状态机中的状态与状态问能够自由转换。但是现实当中的状态却不一定能够自由转换，例如：人可以从站立状态转移到卧倒状态，却不能从卧倒状态直接转移到跑步状态，需要先经过站立状态后再转移到跑步状态。 每个状态可以设置它可以转移到的状态。一些状态机还允许在同一个状态间互相转换，这也需要根据实际情况进行配置。

### 自定义状态需要实现的接口

有限状态机系统需要制定一个状态需具备的属性和功能，由于状态需要由用户自定义，为了统一管理状态，就需要使用接口定义状态。状态机从状态接口查询到用户的自定义状态应该具备的属性有：

除此之外，状态在转移时会发生的事件可以由状态机通过状态接口的方法通知用户自己的状态，对应的是两个方法 OnBegin() 和 OnEnd()，分别代表状态转移前和状态转移后。详细的实现代码如下所示。

```go
package main
import (
    "reflect"
)
// 状态接口
type State interface {
    // 获取状态名字
    Name() string
    // 该状态是否允许同状态转移
    EnableSameTransit() bool
    // 响应状态开始时
    OnBegin()
    // 响应状态结束时
    OnEnd()
    // 判断能否转移到某状态
    CanTransitTo(name string) bool
}
// 从状态实例获取状态名
func StateName(s State) string {
    if s == nil {
        return "none"
    }
    // 使用反射获取状态的名称
    return reflect.TypeOf(s).Elem().Name()
}
```

代码说明如下：

```
第 8 行，声明状态接口。此接口用于状态管理器内部保存和外部实现。
第 14 行，需要实现是否允许本状态间的互相转换。
第 17 和 20 行，需要实现状态的事件，分别是“状态开始”和“状态结束”。当一个状态转移到另外一个状态时，当前状态的 OnEnd() 方法会被调用，而目标状态的 OnBegin() 方法也将被调用。
第 23 行，需要实现本状态能否转移到指定的状态。
第 27 行，通过给定的状态接口查找状态的名称。
```

### 状态基本信息

State 接口中定义的方法，在用户自定义时都是重复的，为了避免重复地编写很多代码，使用 StateInfo 来协助用户实现一些默认的实现。

StateInfo 包含有名称，在状态初始化时被赋值。StateInfo 同时实现了 OnBegin()、OnEnd() 方法。此外，StateInfo 的 EnableSameTransit() 方法还能判断是否允许状态在同类状态中转移，CanTransiTo() 方法能判断是否能转移到某个目标状态，详细实现代码如下所示。

```go
package main
// 状态的基础信息和默认实现
type StateInfo struct {
    // 状态名
    name string
}
// 状态名
func (s *StateInfo) Name() string {
    return s.name
}
// 提供给内部设置名字
func (s *StateInfo) setName(name string) {
    s.name = name
}
// 允许同状态转移
func (s *StateInfo) EnableSameTransit() bool {
    return false
}
// 默认将状态开启时实现
func (s *StateInfo) OnBegin() {
}
// 默认将状态结束时实现
func (s *StateInfo) OnEnd() {
}
// 默认可以转移到任何状态
func (s *StateInfo) CanTransitTo(name string) bool {
    return true
}
```

代码说明如下：

```
第 4 行，声明一个 StateInfo 的结构体，拥有名称的成员。
第 15 行，setName() 方法的首字母小写，表示这个方法只能在同包内被调用。这里我们希望 setName() 不能被使用者在状态初始化后随意修改名称，而是通过后面提到的状态管理器自动赋值。
第 25 和 30 行，对 State 接口的 OnBegin() 和 OnEnd() 方法进行默认实现。
```

### 状态管理

状态管理器管理和维护状态的生命期。用户根据需要，将需要进行状态转移和控制的状态实现后添加（StateManager 的 Add() 方法）到状态管理器里，状态管理器使用名称对这些状态进行维护，同一个状态只允许一个实例存在。

状态管理器可以通过回调函数（StateManager 的 OnChange 成员）提供状态转移的通知。具体状态管理器对状态的管理和维护的代码如下所示。

```go
package main
import "errors"
// 状态管理器
type StateManager struct {
    // 已经添加的状态
    stateByName map[string]State
    // 状态改变时的回调
    OnChange func(from, to State)
    // 当前状态
    curr State
}
// 添加一个状态到管理器
func (sm *StateManager) Add(s State) {
    // 获取状态的名称
    name := StateName(s)
    // 将s转换为能设置名字的接口，然后调用接口
    s.(interface {
        setName(name string)
    }).setName(name)
    // 根据状态名取已经添加的状态，检查是否已经存在
    if sm.Get(name) != nil {
        panic("duplicate state:" + name)
    }
    // 根据名字保存到map中
    sm.stateByName[name] = s
}
// 根据名字获取指定状态
func (sm *StateManager) Get(name string) State {
    if v, ok := sm.stateByName[name]; ok {
        return v
    }
    return nil
}
// 初始化状态管理器
func NewStateManager() *StateManager {
    return &StateManager{
        stateByName: make(map[string]State),
    }
}
```

代码说明如下：

```
第 9 行，声明一个以状态名为键，以 State 接口为值的 map。
第 12 行，状态改变时，状态管理器的成员 OnChange() 函数回调会被调用。
第 15 行，记忆当前状态，当状态改变时，当前状态会变化。
第 22 行，添加状态时，无须提供名称，状态管理器内部会根据 State 的实例和反射查询出状态的名称。
第 25 行，将 s（State 接口）通过类型断言转换为带有 setName() 方法（name string）的接口。接着调用这个接口的 setName() 方法设置状态的名称。使用该方法可以快速调用一个接口实现的其他方法。
第 30 行，根据状态名，在已经添加的状态中检查是否有重名的状态。
第 39 行，根据名称查找状态实例。
第 49 行，构造一个状态管理器。
```

### 在状态间转移

状态管理器不仅管理状态的实例，还可以控制当前的状态及转移到新的状态。状态管理器从当前状态转移到给定名称的状态过程中，如果发现状态不存在、目标状态不能转移及同类状态不能转移时，将返回 error 错误对象，这些错误以 Err 开头，在包（package）里提前定义好。本例一共涉及 3 种错误，分别是：

```
状态没有找到的错误，对应 ErrStateNotFound。
禁止在同状态间转移的错误，对应 ErrForbidSameStateTransit。
不能转移到指定状态的错误，对应 ErrCannotTransitToState。
```

状态转移时，还会调用状态管理器的 OnChange() 函数进行外部通知。状态管理器的状态转移的实现代码如下所示。

```go
// 状态没有找到的错误
var ErrStateNotFound = errors.New("state not found")
// 禁止在同状态间转移
var ErrForbidSameStateTransit = errors.New("forbid same state transit")
// 不能转移到指定状态
var ErrCannotTransitToState = errors.New("cannot transit to state")
// 获取当前的状态
func (sm *StateManager) CurrState() State {
    return sm.curr
}
// 当前状态能否转移到目标状态
func (sm *StateManager) CanCurrTransitTo(name string) bool {
    if sm.curr == nil {
        return true
    }
    // 相同的不用转换
    if sm.curr.Name() == name && !sm.curr.EnableSameTransit() {
        return false
    }
    // 使用当前状态，检查能否转移到指定名字的状态
    return sm.curr.CanTransitTo(name)
}
// 转移到指定状态
func (sm *StateManager) Transit(name string) error {
    // 获取目标状态
    next := sm.Get(name)
    // 目标不存在
    if next == nil {
        return ErrStateNotFound
    }
    // 记录转移前的状态
    pre := sm.curr
    // 当前有状态
    if sm.curr != nil {
        // 相同的状态不用转换
        if sm.curr.Name() == name && !sm.curr.EnableSameTransit() {
            return ErrForbidSameStateTransit
        }
        // 不能转移到目标状态
        if !sm.curr.CanTransitTo(name) {
            return ErrCannotTransitToState
        }
        // 结束当前状态
        sm.curr.OnEnd()
    }
    // 将当前状态切换为要转移到的目标状态
    sm.curr = next
    // 调用新状态的开始
    sm.curr.OnBegin()
    // 通知回调
    if sm.OnChange != nil {
        sm.OnChange(pre, sm.curr)
    }
    return nil
}
```

代码说明如下：

```
第 2～5 行，分别预定义状态转移可能发生的错误。
第 16 行，检查当前状态能否转移到指定名称的状态。
第 32 行，转移到指定状态。
第 43 行，记录转移前的状态，方便在后面代码中通过函数通知外部。
第 46 行，状态管理器初始时，当前状态为 nil，因此无法结束当前状态，只能开始新的状态。
第 49 行，对相同状态的情况进行检查，不能转移时，告知具体错误。
第 54 行，对不能转移的状态，返回具体的错误。
第 59 行，必须要结束当前状态，才能开始新的状态。
```

### 自定义状态实现状态接口

状态的定义和状态管理器的功能已经编写完成，接下来就开始解决具体问题。在解决问题前需要知道有哪些问题：

#### 1) 有哪些状态需要用户自定义及实现？

在使用状态机时，首先需要定义一些状态，并按照 State 状态接口进行实现，以方便自定义的状态能够被状态管理器管理和转移。

本代码定义 3 个状态：闲置（Idle）、移动（Move）、跳跃（Jump）。

#### 2) 这些状态的关系是怎样的？

这 3 个状态间的关系可以通过下图来描述。

<div align=center> 
    <img src="../../img/6-接口/21-示例：实现有限状态机（FSM）/3 个状态间的转移关系.gif"/> 
    <p>图：3 个状态间的转移关系</p>
</div>

3 个状态可以自由转移，但移动（Move）状态只能单向转移到跳跃（Jump）状态。Move 状态可以自我转换，也就是同类状态转移。

状态的转移关系还可以使用表格来描述，如下表所示。

| 下方为当前状态，右方为目标状态 |  Idle 闲置   |  Move 移动   |  Jump 跳跃   |
| :----------------------------: | :----------: | :----------: | :----------: |
|           Idle 闲置            | 同类不能转移 |   允许转移   |   允许转移   |
|           Move 移动            |   允许转移   | 同类允许转移 |   允许转移   |
|           Jump 跳跃            |   允许转移   |  不允许转移  | 同类不能转移 |

#### 3) 如何组织这些状态间的转移？

定义 3 种状态的结构体井内嵌 StateInfo 结构以实现 State 接口中的默认接口。再根据每个状态各自不同的特点，返回状态的转移特点（EnableSameTransit() 及 CanTransitTo() 方法等）及重新实现 OnBegin() 和 OnEnd() 方法的事件回调。详细实现代码如下所示。

```go
// 闲置状态
type IdleState struct {
    StateInfo // 使用StateInfo实现基础接口
}
// 重新实现状态开始
func (i *IdleState) OnBegin() {
    fmt.Println("IdleState begin")
}
// 重新实现状态结束
func (i *IdleState) OnEnd() {
    fmt.Println("IdleState end")
}
// 移动状态
type MoveState struct {
    StateInfo
}
func (m *MoveState) OnBegin() {
    fmt.Println("MoveState begin")
}
// 允许移动状态互相转换
func (m *MoveState) EnableSameTransit() bool {
    return true
}
// 跳跃状态
type JumpState struct {
    StateInfo
}
func (j *JumpState) OnBegin() {
    fmt.Println("JumpState begin")
}
// 跳跃状态不能转移到移动状态
func (j *JumpState) CanTransitTo(name string) bool {
    return name != "MoveState"
}
```

### 使用状态机

3 种自定义状态定义完成后，需要将所有代码整合起来。将自定义状态添加到状态管理器（ StateManager）中，同时在状态改变（StateManager 的 OnChange 成员）时，打印状态转移的详细日志。

在状态转移时，获得转移时可能发生的错误，并且打印错误，详细实现代码如下所示。

```go
package main
import (
    "fmt"
)
func main() {
    // 实例化一个状态管理器
    sm := NewStateManager()
    // 响应状态转移的通知
    sm.OnChange = func(from, to State) {
        // 打印状态转移的流向
        fmt.Printf("%s ---> %s\n\n", StateName(from), StateName(to))
    }
    // 添加3个状态
    sm.Add(new(IdleState))
    sm.Add(new(MoveState))
    sm.Add(new(JumpState))
    // 在不同状态间转移
    transitAndReport(sm, "IdleState")
    transitAndReport(sm, "MoveState")
    transitAndReport(sm, "MoveState")
    transitAndReport(sm, "JumpState")
    transitAndReport(sm, "JumpState")
    transitAndReport(sm, "IdleState")
}
// 封装转移状态和输出日志
func transitAndReport(sm *StateManager, target string) {
    if err := sm.Transit(target); err != nil {
        fmt.Printf("FAILED! %s --> %s, %s\n\n", sm.CurrState().Name(), target, err.Error())
    }
}
```

代码说明如下：

```
第 9 行，创建状态管理器实例。
第 12 行，使用匿名函数响应状态转移的通知。
第 19～21 行，实例化 3 个状态并且添加到管理器。
第 24 行，调用 transitAndReport() 函数，在各种状态间转移。
第 38 行，封装状态转移的过程，并且打印可能发生的错误。
```

运行代码，输出如下：

```
IdleState begin
none ---> IdleState

IdleState end
MoveState begin
IdleState ---> MoveState

MoveState begin
MoveState ---> MoveState

JumpState begin
MoveState ---> JumpState

FAILED! JumpState --> JumpState, forbid same state transit

IdleState begin
JumpState ---> IdleState
```
