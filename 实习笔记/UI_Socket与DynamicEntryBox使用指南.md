# UI Socket 与 `UDynamicEntryBox` 功能和使用指南

> 面向对象：UE5 / UMG 初学者  
> 内容范围：动态条目容器、UI 动态挂载点，以及二者如何配合  
> 脱敏说明：本文中的 Socket 类名、Gameplay Tag、Widget 名称和业务场景均为通用教学示例，不对应任何具体项目。

## 1. 先说结论

`UDynamicEntryBox` 和 UI Socket 不是同一个层级的概念：

- `UDynamicEntryBox` 是 Unreal Engine 的 UMG 模块自带控件，用来创建、排列、移除一组数量会变化的 UserWidget。
- UI Socket 通常不是引擎自带的某个固定类，而是项目在 UMG/CommonUI 之上实现的一种“动态挂载点”架构。
- 一个项目可以让 Socket 继承 `UDynamicEntryBoxBase`，借助它的布局和对象池能力承载动态界面。

可以先把它们理解成：

```text
UDynamicEntryBox
    解决“这些动态生成的 Widget 怎样创建和排列”

UI Socket
    解决“某个 Widget 应该被安装到界面上的哪个位置”
```

一个典型关系如下：

```text
业务功能提供 Widget
        │
        │ 目标标签：UI.Mount.HUD.Health
        ▼
Socket 注册与匹配系统
        │
        │ 找到 HUD 中对应的 Socket
        ▼
UI Socket（动态挂载点）
        │
        │ 使用 DynamicEntryBox 的创建与排列能力
        ▼
实际显示的子 Widget
```

---

## 2. `UDynamicEntryBox` 是什么

`UDynamicEntryBox` 是 UE5 UMG 提供的动态容器控件。它与普通 `Vertical Box`、`Horizontal Box` 的最大区别是：子 Widget 不是主要靠设计器手工放入，而是在运行时根据需要动态创建。

常见用途包括：

- Buff 图标列表；
- 临时通知列表；
- 队伍成员状态条；
- 任务追踪条目；
- 动态快捷键提示；
- 数量较少、需要频繁增删的状态组件。

它的核心工作可以概括为三个步骤：

```text
指定条目 Widget 类型
        ↓
运行时创建或回收条目
        ↓
按照设定的布局方式自动排列
```

### 2.1 类继承关系

```text
UWidget
└── UDynamicEntryBoxBase
    └── UDynamicEntryBox
```

两者职责不同：

| 类型 | 定位 | 适合谁使用 |
|---|---|---|
| `UDynamicEntryBoxBase` | 提供容器、布局和 Widget 池等底层能力 | 希望开发自定义动态容器的程序员 |
| `UDynamicEntryBox` | 提供可直接使用的动态条目创建接口 | 蓝图 UI 和一般业务代码 |

对于入门实践，优先直接使用 `Dynamic Entry Box`。只有在项目需要自己决定“条目从哪里来、怎样匹配、何时创建”时，才考虑继承 `UDynamicEntryBoxBase`。

### 2.2 它和普通 Panel Widget 的区别

普通 `Vertical Box` 的使用方式通常是：

1. 在设计器中放入多个子 Widget；
2. 运行时通过 `Add Child` 或 `Remove Child` 管理；
3. 每个子 Widget 的创建和销毁由业务自行处理。

`UDynamicEntryBox` 的使用方式通常是：

1. 配置统一的条目类 `Entry Widget Class`；
2. 调用 `Create Entry` 让容器创建条目；
3. 调用 `Remove Entry` 或 `Reset` 回收条目；
4. 容器内部使用 Widget Pool 复用对象。

因此，它更像是一个“带布局能力的动态 Widget 工厂”。

---

## 3. `UDynamicEntryBox` 的主要功能

### 3.1 动态创建条目

在蓝图中，最常使用的节点是：

- `Create Entry`：按照 `Entry Widget Class` 创建一个条目；
- `Create Entry of Class`：临时指定一个兼容的条目类来创建；
- `Get All Entries`：获取当前处于使用状态的全部条目；
- `Get Num Entries`：获取当前条目数量。

创建节点会返回新创建或从对象池中复用的 UserWidget。拿到返回值后，应立即设置这一次显示所需的完整数据。

```text
Create Entry
    ↓
转换为 WBP_StatusEntry
    ↓
调用 Initialize Entry
    ↓
设置图标、文本、进度和可见性
```

这里建议给条目 Widget 设计一个统一初始化函数，例如：

```text
InitializeEntry(DisplayName, Icon, Value, MaxValue)
```

不要依赖条目上一次使用时留下的状态，因为对象池可能会把旧对象再次交给你。

### 3.2 移除和清空条目

常用节点包括：

- `Remove Entry`：移除指定条目；
- `Reset`：一次性移除全部条目。

这里的“移除”通常意味着条目离开当前容器，并被放回 Widget Pool 等待复用，不一定会立即销毁 UObject。

这会带来两个重要规则：

1. 不要把“从 Dynamic Entry Box 移除”等同于“对象一定已经销毁”；
2. 每次重新使用条目时，都要重新初始化文本、图片、委托和可见性等状态。

如果条目绑定了外部事件，还应在条目的释放流程中解除绑定，避免重复回调。

### 3.3 自动布局

`Entry Box Type` 决定条目如何排列。常见类型如下：

| 类型 | 效果 | 典型用途 |
|---|---|---|
| Horizontal | 从左到右排列 | 技能栏、Buff 横排 |
| Vertical | 从上到下排列 | 任务列表、通知列表 |
| Wrap | 空间不足时自动换行 | 道具图标、状态图标 |
| Vertical Wrap | 纵向排列后换列 | 特殊图标墙 |
| Overlay | 条目重叠排列 | 层叠装饰、叠加效果 |
| Radial | 按圆形或扇形排列 | 轮盘菜单 |

这些布局方式由 `UDynamicEntryBoxBase` 提供。

需要注意：`Entry Box Type` 更适合作为设计期配置。运行时如果需要完全不同的布局，通常应准备不同容器，而不是频繁切换同一个容器的类型。

### 3.4 条目间距和对齐

常用配置包括：

- `Entry Spacing`：条目间距；
- `Entry Size Rule`：条目在容器中的尺寸规则；
- `Entry Horizontal Alignment`：水平对齐；
- `Entry Vertical Alignment`：垂直对齐；
- `Max Element Size`：某些布局中条目的最大尺寸；
- `Radial Box Settings`：圆形布局参数；
- `Spacing Pattern`：Overlay 布局的间距模式。

初学时可以按下面的经验设置：

- 状态图标横排：Horizontal + 固定间距；
- 通知列表：Vertical + 顶部对齐；
- 可换行图标：Wrap + 合理的最大宽度；
- 轮盘选择：Radial + 明确的起始角度和分布角度。

### 3.5 设计器预览

`Num Designer Preview Entries` 可以在 UMG 设计器里显示若干个预览条目，方便观察间距和布局。

这些预览条目只服务于编辑阶段：

- 它们不是正式业务数据；
- 不代表运行时一定会生成相同数量；
- 不应该被当成运行时条目引用。

---

## 4. `UDynamicEntryBox` 的入门实践

下面以“HUD 状态图标列表”为例。示例名称均为虚构名称。

### 4.1 创建条目 Widget

新建 `WBP_StatusEntry`，包含：

```text
WBP_StatusEntry
└── SizeBox
    └── Overlay
        ├── Image_Icon
        ├── ProgressBar_Duration
        └── Text_Count
```

为它增加一个初始化函数：

```text
InitializeEntry(StatusId, Icon, StackCount, RemainingPercent)
```

初始化函数至少应完成：

- 更新图标；
- 更新层数；
- 更新持续时间；
- 重置不需要显示的控件；
- 保存本次使用对应的数据标识。

### 4.2 在父 Widget 中添加 Dynamic Entry Box

新建 `WBP_StatusBar`，从控件面板搜索并添加 `Dynamic Entry Box`，命名为：

```text
EntryBox_Status
```

在 Details 面板中配置：

| 属性 | 示例值 |
|---|---|
| Entry Widget Class | `WBP_StatusEntry` |
| Entry Box Type | Horizontal |
| Entry Spacing | `(8, 0)` |
| Num Designer Preview Entries | `4` |

### 4.3 添加一个状态

父 Widget 收到“状态新增”事件后：

```text
OnStatusAdded(StatusData)
    ↓
EntryBox_Status → Create Entry
    ↓
得到 WBP_StatusEntry
    ↓
InitializeEntry(StatusData...)
    ↓
把 StatusId 与条目引用记录到 Map
```

建议维护如下映射：

```text
StatusId → EntryWidget
```

这样更新或删除指定状态时不需要遍历全部条目。

### 4.4 更新和删除状态

更新：

```text
根据 StatusId 查找 EntryWidget
    ↓
调用 InitializeEntry 或 RefreshEntry
```

删除：

```text
根据 StatusId 查找 EntryWidget
    ↓
EntryBox_Status → Remove Entry
    ↓
从 Map 删除记录
```

整体刷新：

```text
EntryBox_Status → Reset
    ↓
清空本地 Map
    ↓
按最新数据重新 Create Entry
```

整体刷新写法简单，适合少量条目；高频变化时，优先按 ID 增量更新。

---

## 5. Widget Pool：为什么移除后还能复用

`UDynamicEntryBoxBase` 内部维护 Widget Pool。对象池的目的，是减少频繁创建和销毁 UserWidget 带来的开销。

简化后的生命周期如下：

```text
第一次 Create Entry
        ↓
创建新的 UserWidget
        ↓
显示并使用
        ↓
Remove Entry / Reset
        ↓
进入空闲池
        ↓
下一次 Create Entry
        ↓
从池中取出并重新使用
```

因此，条目 Widget 应遵守“可重复初始化”的原则。

### 5.1 推荐做法

- 使用单一入口函数完整写入显示数据；
- 在重新使用时重设所有可见性状态；
- 重新绑定前先确认旧委托是否已经解除；
- 不在外部长期持有已移除条目的强引用；
- 移除条目后同步清理业务 ID 到 Widget 的映射。

### 5.2 常见错误

假设一个通知条目第一次显示了警告图标，第二次复用时只修改文字，没有隐藏旧图标，那么新通知就可能错误地保留旧图标。

错误思路：

```text
只设置“这一次变化的字段”
```

推荐思路：

```text
每次初始化都把条目恢复为一套完整、确定的状态
```

---

## 6. UI Socket 是什么

UI Socket 可以翻译成“UI 插槽”或“UI 挂载点”。它不是骨骼动画里的 Socket，也不是网络 Socket。

它表示：

> 界面布局提前留出一个带身份标识的位置，其他功能在运行时把自己的 Widget 安装到这个位置。

例如，一个通用 HUD 布局可以只声明：

```text
UI.Mount.HUD.Health
UI.Mount.HUD.Minimap
UI.Mount.HUD.Notification
```

HUD 不需要直接引用具体的生命值面板、小地图或通知面板。业务模块只需要声明“我要挂到哪个标签”，注册系统负责把双方匹配起来。

### 6.1 Socket 解决了什么问题

如果不使用 Socket，HUD 经常会直接引用所有业务 Widget：

```text
HUD
├── 创建生命值 Widget
├── 创建小地图 Widget
├── 创建任务 Widget
├── 创建通知 Widget
└── 知道每个功能何时启用和关闭
```

随着功能增加，根 HUD 会越来越臃肿，布局和业务也高度耦合。

使用 Socket 后：

```text
HUD 布局负责“哪里能放东西”
业务模块负责“要放什么东西”
注册系统负责“把两者配对”
```

这使同一套 HUD 布局更容易被不同模式复用，也便于独立开启或关闭某个 UI 功能。

### 6.2 Socket 的三个核心角色

一个通用 Socket 系统一般包含以下角色：

| 角色 | 职责 | 示例 |
|---|---|---|
| Socket | 声明挂载位置和接收条件 | HUD 上标记为 `UI.Mount.HUD.Health` 的容器 |
| Attachment | 声明要挂载的内容 | `WBP_HealthPanel` 及其目标标签 |
| Registry / Subsystem | 保存注册信息并进行匹配 | 根据标签、上下文和类型完成连接 |

其中 Attachment 可以译为“挂载项”或“附件”。它通常至少包含：

```text
目标 Socket 标签
要创建的 Widget 类或数据对象
所属上下文
显示顺序
可选的匹配条件
```

---

## 7. Socket 的匹配条件

只按名字匹配通常不够。成熟的 Socket 系统一般会组合几个条件。

### 7.1 Gameplay Tag

推荐使用 Gameplay Tag 标识位置，例如：

```text
UI.Mount.HUD.Health
UI.Mount.HUD.Minimap
UI.Mount.HUD.Notification
UI.Mount.Menu.Footer
```

相比字符串，Gameplay Tag 更容易统一管理、检查拼写并表达层级。

### 7.2 精确匹配与层级匹配

精确匹配：

```text
Socket:     UI.Mount.HUD.Notification
Attachment: UI.Mount.HUD.Notification
结果：匹配
```

层级匹配：

```text
Socket:     UI.Mount.HUD
Attachment: UI.Mount.HUD.Notification
结果：可匹配，前提是 Socket 允许匹配子标签
```

初学阶段建议默认使用精确匹配。只有确实需要一个容器接收整类内容时，再启用层级匹配。

### 7.3 上下文 Context

上下文用于隔离同名 Socket，尤其重要的场景是本地多人或分屏。

常见上下文包括：

- 当前 World；
- 当前 LocalPlayer；
- 某个 PlayerController；
- 某个界面实例；
- 某个功能会话对象。

例如两个本地玩家都存在 `UI.Mount.HUD.Health`。如果不带 LocalPlayer 上下文，生命值面板就可能挂到错误玩家的 HUD 上。

### 7.4 接收类型

Socket 还可以限制自己接收的类型：

- 只接收 UserWidget 类；
- 只接收某个基类的 Widget；
- 接收实现指定接口的数据对象；
- 接收数据，再通过工厂转换成 Widget。

类型检查应在创建条目前完成，以便尽早暴露配置错误。

---

## 8. Socket 为什么适合继承 `UDynamicEntryBoxBase`

Socket 自己需要控制条目来源，不能只依赖一个固定的 `Entry Widget Class`。因此，它通常更适合继承 `UDynamicEntryBoxBase`，而不是直接使用 `UDynamicEntryBox`。

```text
UUISocketWidget
└── 继承 UDynamicEntryBoxBase
```

这样可以复用：

- 横向、纵向、换行、叠加、圆形等布局；
- 条目间距和对齐；
- Widget Pool；
- 内部创建、移除和重建能力。

同时由 Socket 子类自己决定：

- 哪些 Attachment 可以进入；
- 每个 Attachment 对应哪个 Widget 类；
- 何时创建和移除；
- 如何保存 Attachment 与条目的对应关系；
- 是否允许多个内容同时挂载。

下面是概念伪代码，类名和接口仅用于说明职责：

```text
class UUISocketWidget : UDynamicEntryBoxBase
    SocketTag
    MatchMode
    Context
    AttachmentToEntryMap

    RegisterSocket()
    UnregisterSocket()
    OnAttachmentMatched(Attachment)
        Entry = CreateEntryInternal(Attachment.WidgetClass)
        AttachmentToEntryMap.Add(Attachment.Id, Entry)

    OnAttachmentRemoved(AttachmentId)
        Entry = AttachmentToEntryMap.Find(AttachmentId)
        RemoveEntryInternal(Entry)
        AttachmentToEntryMap.Remove(AttachmentId)
```

这里的 `CreateEntryInternal` 和 `RemoveEntryInternal` 代表基类提供给派生类使用的底层能力。普通蓝图使用 `UDynamicEntryBox` 时，直接调用公开的 `Create Entry`、`Remove Entry` 即可。

---

## 9. Socket 的完整运行流程

Socket 和 Attachment 谁先出现都应该可以正常工作，因此注册系统通常要同时保存两类记录。

### 9.1 Socket 先注册

```text
HUD 创建
    ↓
Socket 注册标签与上下文
    ↓
注册系统查找当前已有 Attachment
    ↓
没有则等待
    ↓
业务功能稍后注册 Attachment
    ↓
系统匹配 Socket
    ↓
Socket 创建并显示条目
```

### 9.2 Attachment 先注册

```text
业务功能注册 Attachment
    ↓
注册系统暂存 Attachment
    ↓
HUD 稍后创建并注册 Socket
    ↓
系统立即查找已有 Attachment
    ↓
Socket 创建并显示条目
```

如果系统只支持固定的先后顺序，就容易出现加载时序 Bug。一个可靠实现应支持双方以任意顺序注册。

### 9.3 移除流程

```text
业务功能结束
    ↓
使用注册句柄取消 Attachment
    ↓
注册系统通知已匹配的 Socket
    ↓
Socket 移除对应 Entry
    ↓
Entry 返回 Widget Pool
```

因此，注册 Attachment 时最好返回一个 Handle，并由注册者保存：

```text
RegisterAttachment(...) → AttachmentHandle
AttachmentHandle.Unregister()
```

不要只依赖垃圾回收或界面销毁来清理注册关系。

---

## 10. 使用 Socket 搭建 HUD 的入门实践

下面以生命值面板为例，只说明通用流程。

### 10.1 在根 HUD 中放置 Socket

根布局可以是 CommonUI 的 Activatable Widget，也可以是普通 UserWidget。设计器结构示例：

```text
WBP_HUDRoot
└── Overlay_Root
    ├── Socket_Health
    ├── Socket_Minimap
    └── Socket_Notification
```

配置标签：

| Socket 控件 | Socket Tag | 布局方式 |
|---|---|---|
| `Socket_Health` | `UI.Mount.HUD.Health` | Overlay 或 Vertical |
| `Socket_Minimap` | `UI.Mount.HUD.Minimap` | Overlay |
| `Socket_Notification` | `UI.Mount.HUD.Notification` | Vertical |

根 HUD 只声明这些位置，不直接依赖具体业务 Widget。

### 10.2 注册生命值面板

负责玩家状态 UI 的功能在启用时提交一条 Attachment：

```text
TargetTag  = UI.Mount.HUD.Health
WidgetClass = WBP_HealthPanel
Context    = 当前 LocalPlayer
Order      = 0
```

注册系统找到相同标签和上下文的 Socket 后，请求 Socket 创建 `WBP_HealthPanel`。

### 10.3 保存并释放注册句柄

功能对象应保存注册结果：

```text
OnFeatureEnabled
    HealthAttachmentHandle = RegisterAttachment(...)

OnFeatureDisabled
    HealthAttachmentHandle.Unregister()
    HealthAttachmentHandle = Invalid
```

如果忘记注销，可能出现：

- 功能关闭后界面仍然存在；
- 重复启用后出现多个相同 Widget；
- 注册系统持有失效记录；
- 新 HUD 创建时错误恢复旧内容。

### 10.4 让条目接收创建和移除通知

项目可以为挂载内容定义一个通用接口，例如：

```text
OnAddedToSocket(SocketTag, Context)
OnRemovingFromSocket(SocketTag, Context)
```

用途包括：

- 开始或停止监听业务事件；
- 播放进入和退出动画；
- 记录当前挂载位置；
- 解除输入和委托绑定。

如果条目会被对象池复用，这类生命周期通知尤其重要。

### 10.5 编辑器预览与运行时挂载要分开

有些 Socket 控件会提供一个“预览 Widget 类”，让美术在设计器中观察布局。这个预览类只是编辑辅助，不代表运行时一定创建它。

运行时真正挂载的类应来自 Attachment 或正式 UI 配置。

---

## 11. Socket、Dynamic Entry Box 与其他容器怎样选择

| 方案 | 解决的核心问题 | 适合场景 | 不适合场景 |
|---|---|---|---|
| Named Slot | 允许外部向复合控件填入一个设计期内容 | 可复用控件模板 | 多模块运行时注册 |
| Vertical/Horizontal Box | 简单排列手工或运行时添加的子控件 | 固定或少量简单内容 | 需要统一对象池管理 |
| `UDynamicEntryBox` | 创建、复用和排列少量动态条目 | Buff、提示、任务条目 | 数百或数千项大列表 |
| UI Socket | 根据标签和上下文解耦“位置”与“内容” | 模块化 HUD、可插拔功能 UI | 单一局部控件的简单增删 |
| List View / Tile View | 虚拟化显示大量数据项 | 背包、排行榜、长列表 | 少量异构挂载内容 |
| CommonUI Stack / Layer | 管理页面层级、激活、返回和输入 | 菜单页、弹窗页、页面导航 | 单纯的一组状态图标 |

一个简单判断方法：

```text
只需要动态生成一组同类小组件？
    → UDynamicEntryBox

需要让不同功能按标签把内容插入统一 HUD？
    → UI Socket

需要显示大量可滚动数据？
    → List View / Tile View

需要页面入栈、出栈和输入管理？
    → CommonUI Stack / Layer
```

Socket 与 CommonUI 并不冲突。常见组合是：CommonUI 管理页面和输入层级，Socket 管理页面内部的功能挂载，Dynamic Entry Box 负责 Socket 内部条目的创建和排列。

---

## 12. 常见问题与排查

### 12.1 Dynamic Entry Box 没有生成条目

检查：

- 是否设置了 `Entry Widget Class`；
- 是否真正调用了 `Create Entry`；
- 条目类是否是 UserWidget；
- 条目类是否意外引用自身形成递归；
- 容器或父控件是否为 Collapsed；
- 条目尺寸是否为 0；
- 创建逻辑是否只在服务端执行。

### 12.2 条目出现旧数据

原因通常是对象池复用了旧 Widget，而初始化逻辑不完整。

检查：

- 每次创建后是否覆盖全部显示字段；
- 可见性是否重置；
- 动画、计时器和材质参数是否重置；
- 旧事件委托是否解除；
- 旧业务 ID 是否被清除。

### 12.3 条目越用越多

检查：

- 是否重复注册相同 Attachment；
- 功能关闭时是否调用了 Handle 的注销逻辑；
- `AttachmentId → Entry` 映射是否正确删除；
- HUD 重建时旧 Socket 是否注销；
- 是否把“隐藏”误当成“移除”。

### 12.4 Attachment 注册了但没有显示

按以下顺序检查：

1. Socket 和 Attachment 的 Tag 是否完全一致；
2. 双方使用的是精确匹配还是层级匹配；
3. Context 是否为同一个 LocalPlayer 或同一个对象；
4. Socket 是否允许该 Widget 或数据类型；
5. Socket 是否已经完成注册；
6. Attachment 是否被过滤、延迟或取消；
7. 实际创建出的 Widget 是否可见；
8. 父 CommonUI 页面是否已经激活。

### 12.5 分屏时内容挂到了另一个玩家

这通常不是布局问题，而是上下文隔离问题。注册 Socket 和 Attachment 时都应携带对应的 LocalPlayer，并把它作为匹配条件之一。

### 12.6 隐藏 Socket 后输入仍然生效

`Visibility` 只决定显示和命中测试，不等于 CommonUI 的激活状态。

如果挂载内容是 Activatable Widget，还要正确调用激活或停用流程，确保输入映射、返回键和焦点一起释放。

### 12.7 大量条目导致卡顿

`UDynamicEntryBox` 有对象池，但它不是虚拟化列表。它仍可能同时创建并维护所有活动条目。

如果数据量可能达到数百项，应优先考虑：

- `ListView`；
- `TileView`；
- 分页；
- 数据裁剪；
- 只显示屏幕内条目的虚拟化方案。

---

## 13. 推荐的设计约定

### 13.1 Tag 命名

建议按“系统、区域、用途”组织：

```text
UI.Mount.HUD.Health
UI.Mount.HUD.Minimap
UI.Mount.HUD.Notification
UI.Mount.Menu.Header
UI.Mount.Menu.Footer
```

避免：

```text
LeftBox
Slot1
TempArea
```

后者只描述视觉位置，难以表达功能语义，也不利于布局调整。

### 13.2 生命周期

建议明确配对：

```text
RegisterSocket      ↔ UnregisterSocket
RegisterAttachment  ↔ UnregisterAttachment
OnAddedToSocket     ↔ OnRemovingFromSocket
BindEvents          ↔ UnbindEvents
Activate            ↔ Deactivate
```

### 13.3 数据初始化

条目应具备单一、完整、可重复调用的初始化入口。不要让外部依次设置十几个零散字段，否则很容易遗漏对象池遗留状态。

### 13.4 显示顺序

同一 Socket 允许多个 Attachment 时，应定义稳定规则，例如：

```text
先按 Order 从小到大
Order 相同再按注册序号
```

不要依赖容器、Map 或异步加载的偶然遍历顺序。

### 13.5 调试信息

注册系统最好能够输出：

- 当前已注册 Socket；
- 每个 Socket 的 Tag 和 Context；
- 当前已注册 Attachment；
- 匹配成功或失败的原因；
- 每个 Socket 当前创建的条目数量。

这些信息比只打印“创建失败”更容易定位问题。

---

## 14. 最小实践路线

如果你刚接触这套概念，建议按以下顺序练习：

1. 先用 `UDynamicEntryBox` 实现一个动态通知列表；
2. 练习 `Create Entry`、`Remove Entry` 和 `Reset`；
3. 验证条目被复用时，所有显示状态都能正确重置；
4. 再写一个只按 Gameplay Tag 匹配的最小 Socket 注册系统；
5. 增加 Attachment Handle 和注销流程；
6. 增加 LocalPlayer Context，验证分屏隔离；
7. 最后再加入层级匹配、异步加载、显示顺序等能力。

第一阶段不必急着实现非常复杂的全局框架。先让注册、匹配、创建、移除四条路径完整闭环，再逐步扩展。

---

## 15. 总结

记住下面几句话即可：

1. `UDynamicEntryBox` 是 UE 自带的 UMG 控件，适合管理少量、动态变化的 UserWidget 条目。
2. `UDynamicEntryBoxBase` 提供布局和 Widget Pool 等底层能力，`UDynamicEntryBox` 是更适合直接使用的实现。
3. 从容器移除的条目可能进入对象池，所以每次使用都要完整初始化并正确解绑事件。
4. UI Socket 是项目级架构模式，用标签和上下文把“界面位置”与“业务内容”解耦。
5. Socket 可以继承 `UDynamicEntryBoxBase`，复用它的动态创建、回收和排列能力。
6. Socket 系统的关键不是“放进去”这一刻，而是匹配、上下文隔离、注销和生命周期管理。
7. 大数据列表应使用 `ListView` 或 `TileView`，不要因为有对象池就把 `UDynamicEntryBox` 当作虚拟化列表。

延伸阅读：

- [CommonUI 入门指南](./CommonUI_入门指南.md)
- [CommonUI HUD 界面入门实践](./CommonUI_HUD界面入门实践.md)

