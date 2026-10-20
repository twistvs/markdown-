# 模块一：Lua 语言基础（游戏客户端面试）

这一模块主要考察你对 Lua 语言特性的理解。对于 UE5 游戏客户端开发岗位，面试官通常不会只问语法，还会结合 C++、UnLua 追问。

本模块包含以下高频问题：

1. Lua 有哪些基本数据类型？
2. Lua 是动态类型语言还是静态类型语言？
3. Lua 中 nil 和 false 有什么区别？
4. Lua 中的变量和作用域是怎样的？
5. Lua 中的 Table 和 C++ 容器有什么区别？
6. Lua 中的值传递和引用传递是怎样的？
7. Lua 中的 `.` 和 `:` 有什么区别？
8. Lua 中有哪些常用运算符？
9. Lua 的函数有什么特点？
10. Lua 语言与 C++ 相比有什么优缺点？

## 一、Lua 有哪些基本数据类型？

### 1. 面试口头回答

Lua 是一门动态类型语言，一共有 **8 种基本数据类型：**

- `nil`：表示没有有效值。
- `boolean`：布尔类型，只有 `true` 和 `false`。
- `number`：数字类型，用来表示整数或浮点数。
- `string`：字符串类型。
- `table`：表类型，是 Lua 最主要的数据结构。
- `function`：函数类型，Lua 中函数属于一等公民。
- `userdata`：用于 Lua 与 C/C++ 交互，可以表示宿主程序中的数据。
- `thread`：用于实现 Lua 协程。

其中，最重要的是 table、function 和 userdata，因为它们分别涉及 Lua 的数据结构、函数式特性以及与 C++ 的交互。

### 2. 代码示例

```lua
print(type(nil))              -- nil
print(type(true))             -- boolean
print(type(100))              -- number
print(type("hello"))          -- string
print(type({1, 2, 3}))        -- table
print(type(function() end))   -- function
print(type(coroutine.create(
    function() end
)))                           -- thread
```

`userdata` 一般通过 C API 或宿主程序提供，例如在 UnLua 中接触到的某些 C++ 绑定对象。

### 3. 面试追问：number 是否区分 int 和 float？

取决于 Lua 版本。

| Lua 版本           | number 的内部表示                         |
| ------------------ | ----------------------------------------- |
| Lua 5.1 / 5.2      | 标准配置中主要使用 double                 |
| Lua 5.3 / 5.4      | 包含 integer 和 float 两种子类型          |
| LuaJIT（常见配置） | 基于 Lua 5.1 语义，具有自己的数值表示优化 |

以 Lua 5.4 为例：

```
local a = 10
local b = 10.5

print(type(a))       -- number
print(type(b))       -- number

print(math.type(a))  -- integer
print(math.type(b))  -- float
```

虽然内部有整数和浮点数两种表示，但 `type()` 返回的仍然都是 `number`。

![img](../图片/www.lua.png)

Lua

+1



### 4. 面试追问：userdata 是什么？

在普通 Lua 代码中，我们可以直接创建一个 table：

```
local player = {
    hp = 100,
    name = "Warrior"
}
```

但是 Lua 自身不知道 UE5 中的 `AActor`、`UObject` 是什么。

如果希望在 Lua 中访问一个 C++ 对象，绑定框架就需要建立 Lua 与 C++ 对象之间的映射关系。

例如在 UnLua 中可能出现这样的调用：

```
local hp = self:GetHealth()
```

这里 `self` 可能对应一个 UE5 对象的 Lua 绑定表示。

需要注意：并非 userdata 就一定代表一个 C++ 类实例。 它本质上是 Lua 提供给宿主程序使用的一类数据表示机制，具体封装方式由绑定框架决定。

## 二、Lua 是动态类型还是静态类型语言？

高频题

### 1. 面试口头回答

Lua 是动态类型语言。

它的特点是：变量本身不固定属于某种类型，类型由运行时存储的值决定。

因此，同一个变量可以先保存数字，再保存字符串或者 Table，而不需要提前声明类型。

### 2. 代码示例

```
local a = 100
print(type(a))   -- number

a = "hello"
print(type(a))   -- string

a = {hp = 100}
print(type(a))   -- table
```

而在 C++ 中：

```
int a = 100;

// 编译错误
a = "hello";
```

C++ 的变量类型通常在编译期已经确定，Lua 则允许变量在运行过程中关联不同类型的值。

### 3. 面试追问：Lua 是弱类型语言吗？

通常会把 Lua 描述为动态类型语言，但动态类型和弱类型不是同一个概念。

- 动态类型：类型检查和确定主要发生在运行时。
- 弱类型：通常指语言允许一定程度的隐式类型转换，但这个概念没有严格统一的定义。

Lua 在某些操作中允许数字字符串进行隐式转换，但并不是所有类型之间都能随意转换。

所以面试中最准确的回答是：

Lua 是动态类型语言，并且存在有限的隐式类型转换规则。

## 三、Lua 中 nil 和 false 有什么区别？

必会题

这是非常容易出现代码判断题的知识点。

### 1. 面试口头回答

`nil` 和 `false` 是两种不同类型的值。

`nil` 表示没有有效值，通常用于表示变量未赋值，或者 Table 中的某个键不存在。

`false` 是 boolean 类型，表示一个明确的逻辑假值。

Lua 中只有 `nil` 和 `false` 会在条件判断中被视为假值，其余值都被视为真值，包括 `0` 和空字符串。

### 2. 代码示例

```
if 0 then
    print("0 为真")
end

if "" then
    print("空字符串为真")
end

if {} then
    print("空 Table 为真")
end
```

上面三个判断都会成立。

这与 C++ 有很大的区别。

在 C++ 中：

```
if (0) {
    // 不会执行
}
```

在 Lua 中：

```
if 0 then
    -- 会执行
end
```

### 3. nil 和 false 在 Table 中的区别

```
local t = {}

t["a"] = false
t["b"] = nil

print(t["a"])  -- false
print(t["b"])  -- nil
```

虽然下面两个条件都不成立：

```
if t["a"] then
    print("a")
end

if t["b"] then
    print("b")
end
```

但是本质不同：

- `t["a"] = false`：键 `a` 存在，对应的值是 false。
- `t["b"] = nil`：相当于移除键 `b`。

因此，当 Table 的字段有可能存储 `false` 时，不能简单地通过 `if t[key]` 判断这个字段是否存在。

可以用：

```
if t["a"] ~= nil then
    print("字段存在且不为 nil")
end
```

### 4. 面试追问：下面代码输出什么？

```
local a = false
local b = 0
local c = ""

print(a or 100)
print(b or 100)
print(c or 100)
```

查看答案

这一点也解释了为什么 Lua 中使用：

```
local hp = inputHp or 100
```

作为默认值时，如果 `inputHp` 是 `false`，也会得到 `100`。

![img](../图片/www.lua.png)

Lua

+1

## 四、Lua 中的变量和作用域

### 1. 面试口头回答

Lua 中变量主要分为局部变量、全局变量和 Table 字段。

其中，使用 `local` 声明的是局部变量，作用域通常是当前代码块；没有使用 `local` 声明的普通变量名，默认按照全局变量规则访问。

Lua 的局部变量采用词法作用域，因此内部函数可以访问外部函数的局部变量，这也是 Lua 实现闭包的基础。

### 2. 全局变量与局部变量

```
a = 100           -- 全局变量
local b = 200     -- 局部变量

function Test()
    local c = 300
    print(a)      -- 100
    print(b)      -- 200
    print(c)      -- 300
end

Test()

print(c)          -- nil
```

在普通 Lua 5.1 全局环境中，全局变量可以通过 `_G` 访问：

```
hp = 100

print(_G.hp)    -- 100
```

Lua 5.2 之后使用 `_ENV` 机制来解析全局名称，虽然通常也能访问 `_G`，但实现原理有所变化。

### 3. 为什么游戏开发中建议尽量使用 local？

主要有三个原因：

1. 避免不同 Lua 模块之间出现全局变量名称冲突。
2. 减少对全局环境的依赖，提高代码可维护性。
3. 局部变量在某些访问场景下通常有更好的性能表现。

比如：

```
-- 不推荐
PlayerHP = 100

-- 推荐
local PlayerHP = 100
```

如果是较大的 Lua 游戏项目，随意使用全局变量可能造成难以定位的状态修改。

## 五、Lua 中的 Table 有什么特点？

这一题是模块二的重点，这里先掌握基础。

### 1. 面试口头回答

Table 是 Lua 中唯一内建的通用数据结构机制，能够用来实现数组、字典、集合以及面向对象编程中的对象。

与 C++ 的 `vector` 不同，Table 不要求元素类型相同，也不要求键必须为整数，可以使用字符串、数字等多种类型作为键。

在底层实现上，标准 Lua 的 Table 通常由数组部分和哈希部分共同组成，用于处理不同形式的键。

### 2. 代码示例

```
local player = {
    name = "Warrior",
    hp = 100,
    skills = {"Attack", "Dash"}
}

print(player.name)       -- Warrior
print(player["hp"])      -- 100
print(player.skills[1])  -- Attack
```

注意：Lua 中习惯从索引 1 开始使用数组。

```
local arr = {10, 20, 30}

print(arr[1])    -- 10
print(arr[0])    -- nil
```

但是 Lua 并不禁止使用 `0` 或负数作为 Table 的键。

### 3. Table 的遍历

```
local arr = {10, 20, 30}

for i, v in ipairs(arr) do
    print(i, v)
end
```

对于字典类型：

```
local player = {
    name = "Warrior",
    hp = 100
}

for k, v in pairs(player) do
    print(k, v)
end
```

区别：

- `ipairs`：从索引 1 开始依次遍历，遇到第一个 nil 就停止。
- `pairs`：遍历 Table 的键值对，不保证遍历顺序。

### 4. 面试追问：#table 一定能得到元素数量吗？

不一定。

```
local t = {10, 20, 30}
print(#t)  -- 3
```

但是：

```
local t = {
    [1] = 10,
    [3] = 30
}
```

此时由于数组中存在空洞，`#t` 不能被当作稳定、可靠的元素总数。

对于存在空洞的序列，长度操作符返回的是满足特定条件的边界，而不是非 nil 元素的数量。

## 六、Lua 中的值传递和引用传递

必会题

这道题很容易与 C++ 中的指针、引用机制结合起来考察。

### 1. 面试口头回答

Lua 的赋值和函数传参可以统一理解为值传递。

但对于不同数据类型，传递的值有所不同。

对于 number、boolean 等类型，变量保存的是对应的值；而对于 Table、function、thread、userdata 等对象类型，变量保存的是对象引用。

所以，当两个变量指向同一个 Table 时，通过一个变量修改 Table 的内容，另一个变量也能观察到修改。

### 2. number 类型

```
local a = 10
local b = a

b = 20

print(a)  -- 10
print(b)  -- 20
```

`b` 的重新赋值不会影响 `a`。

### 3. Table 类型

```
local a = {hp = 100}
local b = a

b.hp = 50

print(a.hp)  -- 50
print(b.hp)  -- 50
```

引用关系示意图

变量 a 和 b 都引用同一个 Table 对象。

### 4. 面试追问：将 Table 传入函数，在函数中修改 Table，一定会影响外部吗？

不一定，要看修改的是什么。

```
function Modify(t)
    t.hp = 50
end

local player = {hp = 100}

Modify(player)

print(player.hp)  -- 50
```

这里修改的是对象内部字段，因此外部受到影响。

但是：

```
function Modify(t)
    t = {hp = 50}
end

local player = {hp = 100}

Modify(player)

print(player.hp)  -- 100
```

这里仅仅将函数内部的局部形参 `t` 重新绑定到了另一个 Table，并没有改变外部变量引用的对象。

这是非常典型的面试陷阱。

## 七、Lua 中的 `.` 和 `:` 有什么区别？

必会题

这个知识点在 UnLua 开发中很重要。

### 1. 面试口头回答

Lua 中的 `.` 用于访问 Table 的字段或函数，而 `:` 是方法调用和方法定义的语法糖。

通过 `:` 调用方法时，Lua 会自动把调用者作为第一个参数 `self` 传进去。

因此，两者最重要的区别就是是否自动传递 `self`。

### 2. 代码示例

```
local Player = {}

function Player.Attack(self)
    print("Attack")
end

Player.Attack(Player)
```

可以简写成：

```
local Player = {}

function Player:Attack()
    print("Attack")
end

Player:Attack()
```

两种写法效果相同。

可以理解为：

```
Player:Attack()

-- 等价于
Player.Attack(Player)
```

### 3. 面试追问：下面代码有什么问题？

```
local Player = {}

function Player:Attack()
    print(self.hp)
end

Player.hp = 100

Player.Attack()
```

问题在于使用 `.` 调用时没有显式传递 `self`。

函数内部访问 `self.hp` 时，`self` 是 nil，运行时会报错。

正确写法：

```
Player:Attack()

-- 或者
Player.Attack(Player)
```

### 4. 联系 UE5 / UnLua

在 UnLua 中经常使用：

```
function M:ReceiveBeginPlay()
    self.Super.ReceiveBeginPlay(self)
end
```

这里的 `self` 通常代表当前绑定的 UE 对象。

`self.Super.ReceiveBeginPlay(self)` 使用点号访问函数，因此显式传入了 `self`。具体父类调用写法仍需遵循所使用的 UnLua 版本及其 API 约定。

## 八、Lua 常见运算符

主要记住与 C++ 不同的部分。

| 功能       | Lua   | C++                      |
| ---------- | ----- | ------------------------ |
| 不等于     | `~=`  | `!=`                     |
| 逻辑与     | `and` | `&&`                     |
| 逻辑或     | `or`  | `\|\|`                   |
| 逻辑非     | `not` | `!`                      |
| 字符串拼接 | `..`  | `+` 或其他方式           |
| 取长度     | `#`   | 根据容器使用 `size()` 等 |

### 1. and 和 or 不一定返回 boolean

```
print(10 and 20)     -- 20
print(10 or 20)      -- 10
print(nil and 20)    -- nil
print(nil or 20)     -- 20
```

`and` 和 `or` 都具有短路求值特性，并且会返回某个操作数的值。

### 2. Table 默认比较对象身份

```
local a = {1, 2, 3}
local b = {1, 2, 3}
local c = a

print(a == b)  -- false
print(a == c)  -- true
```

虽然 `a` 和 `b` 的内容相同，但它们是两个不同的对象，因此默认情况下不相等。

后面学习元表时，我们会讲到如何通过 `__eq` 自定义对象比较行为。

## 九、Lua 中函数有什么特点？

### 1. 面试口头回答

Lua 中函数是一等公民，也就是说，函数可以像普通值一样被赋值给变量、作为参数传入其他函数，或者作为其他函数的返回值。

Lua 还支持匿名函数、可变参数、多返回值和闭包。

### 2. 函数可以赋值

```
local function Add(a, b)
    return a + b
end

local f = Add

print(f(10, 20))  -- 30
```

### 3. 函数可以作为参数

```
function Execute(callback)
    callback()
end

Execute(function()
    print("执行回调")
end)
```

在游戏客户端开发中，这类写法可以用来处理事件回调、UI 回调和异步逻辑的结果通知。

### 4. 函数支持多个返回值

```
function GetPosition()
    return 10, 20, 30
end

local x, y, z = GetPosition()

print(x, y, z)  -- 10 20 30
```

与常见的 C++ 返回单一值的函数不同，Lua 原生支持一次返回多个结果。

### 5. 函数支持可变参数

```
function PrintArgs(...)
    local args = {...}

    for i, v in ipairs(args) do
        print(i, v)
    end
end

PrintArgs(10, 20, 30)
```

这里的 `...` 表示可变参数。

对于可能包含 nil 的参数，不应该单纯依赖 `{...}` 和 `#` 来确定参数个数，应使用 `select("#", ...)`。

## 十、Lua 和 C++ 有什么区别？为什么游戏客户端使用 Lua？

综合题

### 1. 面试口头回答

Lua 和 C++ 的主要区别是，C++ 通常是静态编译型语言，而 Lua 一般作为嵌入式脚本语言，通过 Lua 虚拟机执行代码。

C++ 的优势在于运行性能、内存控制和底层系统开发能力，因此适合实现游戏引擎、渲染、物理计算以及性能敏感的逻辑。

Lua 的优势是轻量、灵活，并且比较容易嵌入 C++ 程序中。Lua 脚本还可以在一定的工程条件下实现热更新，因此适合开发更新频繁的业务逻辑。

在 UE5 项目中，可以使用 UnLua 连接 Lua 和 C++，把底层能力放在 C++ 中实现，把部分游戏业务逻辑放在 Lua 中实现，从而兼顾性能和迭代效率。

### 2. 对比表

| 比较维度 | Lua                                  | C++                              |
| -------- | ------------------------------------ | -------------------------------- |
| 类型系统 | 动态类型                             | 静态类型                         |
| 执行方式 | 通常由解释器或虚拟机执行             | 通常编译为机器码                 |
| 内存管理 | 主要通过 GC                          | RAII、智能指针、手动管理等       |
| 性能     | 取决于实现，一般不及优化后的原生 C++ | 通常更适合性能敏感逻辑           |
| 热更新   | 脚本层相对容易实现                   | 原生代码更新通常更复杂           |
| 主要用途 | 业务逻辑、脚本、配置等               | 引擎底层、核心系统、性能敏感逻辑 |

这里还需要强调：使用 Lua 并不意味着所有游戏逻辑都应该放在 Lua 中。

例如，游戏中每帧需要处理数十万个实体的密集计算，通常更适合在 C++ 等高性能实现中完成。

## 十一、模块一面试自测

下面是 5 道接近真实技术面试的基础题，可以自行作答，再查看参考答案。

1. Lua 中只有 false 会被判断为假值吗？

答案

2. Lua 中 local a = {}，然后 local b = a。修改 b 会影响 a 吗？

答案

3. Lua 的 Table 是不是所有键都从 1 开始？

答案

4. Player:Attack() 和 Player.Attack() 完全相同吗？

答案

5. Lua 中传递 Table 给函数，是 C++ 意义上的引用传参吗？

答案

## 本模块掌握标准

对于 UE5 游戏客户端技术面试，我建议重点掌握以下五个核心概念：

动态类型、nil 与 false、Table 的引用语义、local 作用域、`.` 与 `:` 的区别。

其中 Table 的引用语义和方法调用语法是游戏开发中非常实用的知识点，也最容易结合 UnLua 被追问。

下一模块是 Lua Table 的底层实现，会重点讲解数组部分与哈希部分、扩容、哈希冲突以及为什么 Lua 用 Table 就能够实现数组、字典和面向对象。