# MVVM 入门指南

> 适合第一次接触 MVVM 的开发者。本文先解释核心思想，再通过一个简单的“待办事项”示例理解各层职责。

## 1. MVVM 是什么

MVVM 是 **Model–View–ViewModel** 的缩写，是一种用于组织界面应用代码的架构模式。

它希望解决的核心问题是：**不要把界面显示、业务数据和交互逻辑全部写在一起。**

MVVM 将代码分成三部分：

| 部分 | 中文理解 | 主要职责 |
| --- | --- | --- |
| Model | 数据与业务模型 | 保存数据、执行业务规则、访问接口或数据库 |
| View | 界面 | 展示内容、接收用户操作 |
| ViewModel | 界面模型 | 为 View 准备数据和命令，协调 View 与 Model |

最简化的数据流如下：

```text
用户操作
   ↓
 View  ──调用命令──>  ViewModel  ──调用──>  Model
   ↑                      │                    │
   └────── 数据绑定 ──────┘ <────返回数据─────┘
```

## 2. 为什么使用 MVVM

如果一个页面中的按钮事件、网络请求、数据处理和界面更新都写在 View 中，代码会很快变得难以阅读和测试。

MVVM 的主要好处包括：

- **职责清晰**：界面、状态和业务数据分别管理。
- **便于测试**：ViewModel 通常不依赖具体界面，可以单独测试。
- **便于维护**：修改界面时，不必大范围改动业务逻辑。
- **便于复用**：同一个 Model 或 ViewModel 可以服务于不同界面。
- **减少手动刷新界面**：通过数据绑定，让界面随状态自动变化。

MVVM 也不是免费的。简单页面如果拆分得过细，可能增加文件数量和理解成本。因此，应根据项目复杂度合理使用。

## 3. 三个核心角色

### 3.1 Model：数据和业务规则

Model 表示应用中的数据以及与这些数据直接相关的业务行为。

例如，一个待办事项可以表示为：

```text
TodoItem
├── Id
├── Title
└── IsCompleted
```

Model 层还可能包含：

- 从服务器读取待办事项；
- 将待办事项保存到数据库；
- 校验标题是否为空；
- 根据业务规则计算结果。

Model 不应该关心：

- 页面上用了按钮还是菜单；
- 文本显示成什么颜色；
- 某个控件位于屏幕的哪个位置。

### 3.2 View：用户看到的界面

View 负责展示数据并接收用户输入，例如：

- 文本框；
- 按钮；
- 列表；
- 加载动画；
- 错误提示区域。

View 应尽量只描述“界面是什么样”和“绑定什么数据”，不要承担复杂业务逻辑。

可以把 View 理解为一个观察者：它观察 ViewModel 的状态，并将最新状态展示给用户。

### 3.3 ViewModel：连接 View 和 Model

ViewModel 是 MVVM 中最关键的一层。它把 Model 中的数据转换为 View 容易使用的状态，同时提供可由 View 触发的操作。

例如，待办事项页面的 ViewModel 可能包含：

```text
TodoListViewModel
├── NewTodoTitle        新待办标题
├── Todos               待办列表
├── IsLoading           是否正在加载
├── ErrorMessage        错误信息
├── AddTodoCommand      添加命令
└── LoadTodosCommand    加载命令
```

ViewModel 应该知道页面“需要什么状态”，但一般不应该直接操作具体控件。例如，不要在 ViewModel 中查找某个按钮并修改按钮文字，而应修改一个属性，再由 View 通过绑定显示它。

## 4. MVVM 的两个关键机制

### 4.1 数据绑定（Data Binding）

数据绑定用于连接 View 中的控件和 ViewModel 中的属性。

例如：

```text
ViewModel.NewTodoTitle  ←→  View.标题输入框
ViewModel.Todos         ─→  View.待办列表
ViewModel.IsLoading     ─→  View.加载动画是否显示
```

常见绑定方向：

| 类型 | 数据方向 | 适用场景 |
| --- | --- | --- |
| 单向绑定 | ViewModel → View | 显示文本、列表、加载状态 |
| 双向绑定 | ViewModel ↔ View | 输入框、开关、选择项 |
| 单次绑定 | ViewModel → View，仅初始化一次 | 不再变化的静态信息 |

不同技术栈的名称和语法可能不同，但思想相同：**ViewModel 的状态变化后，View 能够得到通知并更新。**

### 4.2 命令（Command）

命令用于把用户操作交给 ViewModel。

```text
用户点击“添加”按钮
        ↓
View 触发 AddTodoCommand
        ↓
ViewModel 校验输入并调用 Model
        ↓
ViewModel 更新 Todos 或 ErrorMessage
        ↓
View 通过绑定自动刷新
```

命令通常包含两类能力：

- **执行操作**：点击后要做什么；
- **判断能否执行**：当前按钮是否应该可用。

例如，当标题为空时，`AddTodoCommand` 可以处于不可执行状态，View 中的“添加”按钮随之禁用。

## 5. 一个完整的小例子

下面使用与具体语言无关的伪代码，演示一个待办事项页面。

### 5.1 Model

```text
class TodoItem:
    Id
    Title
    IsCompleted

class TodoRepository:
    function GetAll():
        return 从本地或服务器读取待办列表

    function Add(title):
        校验并保存新待办
        return 新建的 TodoItem
```

### 5.2 ViewModel

```text
class TodoListViewModel:
    NewTodoTitle = ""
    Todos = []
    IsLoading = false
    ErrorMessage = ""

    function LoadTodos():
        IsLoading = true
        ErrorMessage = ""

        try:
            Todos = repository.GetAll()
        catch error:
            ErrorMessage = "加载失败，请稍后重试"
        finally:
            IsLoading = false

    function CanAddTodo():
        return NewTodoTitle 去除空格后不为空

    function AddTodo():
        if not CanAddTodo():
            return

        newItem = repository.Add(NewTodoTitle)
        Todos.Add(newItem)
        NewTodoTitle = ""
```

### 5.3 View

```text
输入框.Text       双向绑定 NewTodoTitle
添加按钮.Command 绑定 AddTodoCommand
待办列表.Items    单向绑定 Todos
加载动画.Visible  单向绑定 IsLoading
错误区域.Text     单向绑定 ErrorMessage
```

在这个例子中，View 不负责保存待办事项，也不决定错误时如何组织页面状态。它只负责绑定和展示。

## 6. 一次交互是怎样流动的

以“添加待办事项”为例：

1. 用户在 View 的输入框中输入标题。
2. 双向绑定把标题写入 ViewModel 的 `NewTodoTitle`。
3. 用户点击“添加”按钮。
4. View 触发 ViewModel 的 `AddTodoCommand`。
5. ViewModel 校验输入并调用 Model 或 Repository 保存数据。
6. ViewModel 更新 `Todos` 和 `NewTodoTitle`。
7. 属性变化通知触发绑定更新。
8. View 显示新的列表，并清空输入框。

如果你能清楚地说出某个交互在这三层之间如何流动，就已经抓住了 MVVM 的核心。

## 7. ViewModel 中常见的页面状态

一个实用的 ViewModel 往往不只包含业务数据，还包含页面状态：

```text
IsLoading       是否加载中
IsEmpty         是否没有数据
ErrorMessage    错误信息
SelectedItem    当前选中项
SearchKeyword   搜索关键字
CanSubmit       是否允许提交
```

注意：这些是“界面需要的数据”，但不等于“具体控件”。

较好的写法：

```text
IsLoading = true
```

不推荐的写法：

```text
LoadingSpinner.Show()
```

前者表达状态，由 View 决定如何显示；后者让 ViewModel 直接依赖具体控件。

## 8. 常见误区

### 8.1 把 ViewModel 当成万能层

不要把所有逻辑都塞进 ViewModel。复杂业务规则应放入 Model、Service 或领域层，数据访问应交给 Repository 或类似组件。

### 8.2 ViewModel 直接操作控件

ViewModel 如果持有文本框、按钮或窗口对象，就会与具体界面耦合，降低可测试性和复用性。

### 8.3 Model 只是没有行为的数据结构

Model 可以包含业务行为和校验规则，并不一定只是字段集合。关键是它不依赖具体界面。

### 8.4 所有状态都使用双向绑定

只有确实需要由 View 写回 ViewModel 的内容才使用双向绑定。显示列表、加载状态等通常使用单向绑定即可。

### 8.5 在 ViewModel 中弹窗或跳转页面

弹窗和页面导航通常属于界面行为。常见做法是通过接口、事件或导航服务表达需求，避免 ViewModel 直接创建具体窗口。

### 8.6 忽略异步状态和异常

请求数据时，应考虑加载中、成功、空数据、失败和取消等状态，避免只处理“成功路径”。

## 9. View 的代码必须完全为空吗

不必。

View 可以保留纯界面相关的代码，例如：

- 动画控制；
- 焦点切换；
- 拖拽；
- 复杂控件的视觉交互；
- 与平台 UI 生命周期直接相关的处理。

判断标准是：这段代码是否只关心“怎样显示和交互”，并且不包含核心业务规则。

## 10. MVVM 与 MVC、MVP 的简单区别

| 模式 | 中间角色 | View 更新方式（典型情况） | 主要特点 |
| --- | --- | --- | --- |
| MVC | Controller | Controller 协调 Model 和 View | Web 和传统应用中常见 |
| MVP | Presenter | Presenter 主动调用 View 接口 | View 通常较被动 |
| MVVM | ViewModel | View 通过数据绑定观察 ViewModel | 强调可绑定状态和命令 |

实际框架可能混合多种思想，不必过分纠结名称。更重要的是保持职责清晰和依赖方向合理。

## 11. 如何开始实践

建议从一个小页面开始：

1. 列出页面要展示和编辑的状态。
2. 将这些状态定义为 ViewModel 的属性。
3. 将按钮操作定义为 ViewModel 的命令或方法。
4. 把持久化和网络访问放入 Model、Service 或 Repository。
5. 在 View 中建立绑定。
6. 为 ViewModel 编写不依赖真实界面的测试。

一个合适的练习项目是“待办事项”：

- 显示待办列表；
- 添加、删除和完成待办；
- 支持加载状态和错误信息；
- 增加搜索或筛选；
- 使用模拟 Repository 测试 ViewModel。

## 12. ViewModel 测试思路

测试 ViewModel 时，重点验证输入、操作和状态结果，不需要真正打开页面。

例如：

```text
测试：标题为空时不能添加
Given  NewTodoTitle = "   "
When   查询 AddTodoCommand 是否可执行
Then   结果为 false

测试：添加成功后清空输入框
Given  NewTodoTitle = "学习 MVVM"
When   执行 AddTodoCommand
Then   Todos 中新增一项
And    NewTodoTitle = ""

测试：加载失败时显示错误状态
Given  Repository 被设置为返回错误
When   执行 LoadTodosCommand
Then   IsLoading = false
And    ErrorMessage 不为空
```

## 13. 初学者检查清单

编写一个 MVVM 页面后，可以检查：

- [ ] View 是否主要负责展示和绑定？
- [ ] ViewModel 是否不直接依赖具体控件？
- [ ] Model 或 Service 是否承担了数据和业务规则？
- [ ] 用户操作是否通过命令或清晰的方法进入 ViewModel？
- [ ] 状态变化是否能通知 View？
- [ ] 是否区分了加载、成功、空数据和失败状态？
- [ ] 是否避免了不必要的双向绑定？
- [ ] ViewModel 是否可以在不启动界面的情况下测试？

## 14. 一句话总结

MVVM 的核心不是“必须创建三个文件”，而是：

> View 负责展示，Model 负责数据与业务，ViewModel 用可绑定的状态和命令连接二者。

学习时先掌握 **职责划分、数据绑定、命令和状态变化通知**，再结合具体框架学习语法，会更容易理解。

