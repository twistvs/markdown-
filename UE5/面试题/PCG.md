下面介绍 **Unreal Engine 5 的 PCG，Procedural Content Generation，程序化内容生成框架**。

## 1. PCG 是什么？

**PCG 是 UE5 中用于程序化生成场景内容的节点式工具框架**。

它可以根据规则自动生成大量内容，例如：

```text
树木、草地、岩石
道路两侧的路灯
山坡上的植被
建筑周围的杂物
围栏、管线、战壕
城市街区道具
敌方营地布置
大型战场环境细节
```

Epic 官方文档对 PCG 的定位是：在 Unreal Engine 内部创建程序化内容和程序化工具的工具集，艺术家和设计师可以通过节点图来快速生成内容。([Epic Games Developers](https://dev.epicgames.com/documentation/unreal-engine/procedural-content-generation-framework-in-unreal-engine?utm_source=chatgpt.com))

简单理解：

```text
手工摆放：
一个一个放树、石头、草、建筑道具

PCG：
制定规则，让系统自动生成、分布、筛选、替换、更新
```

------

## 2. PCG 解决了什么问题？

在大型场景中，手工摆放内容会遇到几个问题：

- 工作量巨大；
- 内容重复但又需要变化；
- 场景修改后需要重新手动调整；
- 开放世界、山地、森林、城市、战场等场景细节难以维护；
- 多人协作时很容易出现风格不一致。

PCG 的目标是：

> 让你把“摆放内容”变成“设计规则”。

例如你可以定义：

```text
在坡度小于 25° 的地面生成草
在海拔 1200m 以上减少乔木
在道路两侧每隔 8m 放一盏路灯
在建筑周围 2m 内不要生成树
在敌方营地中心附近生成车辆、沙袋、雷达和巡逻点
```

这样场景地形或道路变化后，PCG 可以根据规则重新生成内容。

------

## 3. PCG 的核心工作方式

PCG 通常使用 **PCG Graph**，也就是程序化生成节点图。

一个典型流程是：

```text
输入数据 → 采样点 → 筛选点 → 修改属性 → 生成对象
```

例如生成森林：

```text
Landscape 输入
↓
Surface Sampler 在地表采样点
↓
Slope Filter 根据坡度过滤
↓
Density Filter 根据密度过滤
↓
Transform Points 随机旋转和缩放
↓
Static Mesh Spawner 生成树木、石头、灌木
```

PCG 不是单纯“随机撒点”，而是**基于规则、空间、属性和数据的内容生成系统**。

------

## 4. PCG 的几个核心概念

### 4.1 PCG Graph

**PCG Graph 是 PCG 的核心资产**。

它类似材质编辑器或蓝图，由多个节点连接组成。

你可以在里面设计：

- 从哪里取数据；
- 如何采样；
- 如何过滤；
- 如何随机化；
- 如何生成 Actor 或 Mesh；
- 如何输出结果。

官方节点参考文档说明，PCG 框架使用 Procedural Node Graph，可以在编辑器和运行时生成程序化内容。([Epic Games Developers](https://dev.epicgames.com/documentation/unreal-engine/procedural-content-generation-framework-node-reference-in-unreal-engine?utm_source=chatgpt.com))

------

### 4.2 PCG Component

**PCG Component 是挂在 Actor 上的组件**。

一般流程是：

```text
创建一个 Actor
添加 PCG Component
指定 PCG Graph
设置生成范围
执行 Generate
```

你可以把一个 PCG Graph 当成“生成规则”，把 PCG Component 当成“规则执行器”。

------

### 4.3 Points 点数据

PCG 中最重要的数据之一是 **Point**。

每个 Point 通常包含：

```text
位置 Location
旋转 Rotation
缩放 Scale
密度 Density
颜色 Color
种子 Seed
自定义属性 Attribute
```

大多数 PCG 逻辑本质上都是在处理点：

```text
生成点 → 过滤点 → 修改点 → 用点生成物体
```

例如一个点可以代表：

- 一棵树的位置；
- 一块石头的位置；
- 一个敌人出生点；
- 一个路灯位置；
- 一个建筑模块位置。

------

### 4.4 Bounds / Volume 生成范围

PCG 通常需要一个范围来决定在哪里生成。

范围可以来自：

- PCG Volume；
- Actor Bounds；
- Landscape；
- Spline；
- Static Mesh；
- 自定义输入数据。

例如你可以画一个 PCG Volume，让它只在这个区域内生成森林。

------

### 4.5 Attribute 属性

Attribute 是 PCG 变强的关键。

你可以给点附加属性，例如：

```text
Biome = Alpine
Slope = 18
Altitude = 2350
RoadDistance = 4.5
EnemyCampLevel = 2
VegetationType = Shrub
```

然后根据属性决定生成什么。

例如：

```text
如果 Altitude > 3000：
    生成低矮灌木和岩石

如果 RoadDistance < 3：
    不生成大树

如果 Biome = Snow：
    使用雪地岩石和冷杉
```

------

## 5. PCG 常见节点类型

### 5.1 输入节点

用于获取场景数据。

常见输入包括：

```text
Landscape
Spline
Actor
Static Mesh
Volume
World Ray Hit Query
```

例如用 Landscape 获取地形表面，用 Spline 获取道路路径。

------

### 5.2 采样节点

用于生成点。

常见例子：

```text
Surface Sampler：在表面采样点
Spline Sampler：沿样条线采样点
Volume Sampler：在体积内采样点
Mesh Sampler：在模型表面采样点
```

例如沿一条道路 Spline 每隔固定距离采样点，然后生成电线杆。

------

### 5.3 过滤节点

用于删除不符合条件的点。

常见过滤条件：

```text
坡度
高度
密度
距离
标签
属性
碰撞
随机概率
```

例如：

```text
坡度太陡的点删除
离道路太近的点删除
海拔太低的点删除
建筑范围内的点删除
```

------

### 5.4 变换节点

用于修改点的位置、旋转、缩放。

例如：

```text
随机旋转
随机缩放
沿法线对齐
偏移位置
投射到地面
```

这样生成出来的树、石头不会显得整齐重复。

------

### 5.5 生成节点

用于把点转换为实际内容。

常见输出包括：

```text
Static Mesh Spawner
Actor Spawner
Foliage Spawner
Instanced Static Mesh
Hierarchical Instanced Static Mesh
```

大型场景中通常优先考虑 Instanced Static Mesh 或 HISM，以降低 Draw Call 和内存开销。

------

## 6. PCG 和 Foliage 的区别

| 对比项             | Foliage 工具    | PCG                                    |
| ------------------ | --------------- | -------------------------------------- |
| 核心用途           | 快速刷植被      | 规则化生成各种内容                     |
| 控制方式           | 手刷 + 基本规则 | 节点图规则                             |
| 可扩展性           | 较有限          | 很强                                   |
| 适合内容           | 草、树、石头    | 植被、道路设施、建筑模块、营地、任务点 |
| 是否容易复用       | 一般            | 很容易复用                             |
| 是否能根据属性生成 | 较弱            | 很强                                   |
| 程序化逻辑         | 简单            | 复杂                                   |

可以这样理解：

```text
Foliage 更像画笔；
PCG 更像规则系统。
```

如果只是快速铺草，Foliage 很方便。

如果你要做“根据地形、道路、区域、任务状态自动布置环境”，PCG 更合适。

------

## 7. PCG 和蓝图的区别

PCG 与蓝图不是替代关系，而是互补关系。

| 对比项   | Blueprint                 | PCG                            |
| -------- | ------------------------- | ------------------------------ |
| 强项     | 逻辑、交互、事件、玩法    | 空间分布、批量生成、环境构建   |
| 数据流   | 执行流                    | 数据流 / 点流                  |
| 典型用途 | 门开关、任务系统、AI 行为 | 树木、岩石、道路设施、场景道具 |
| 可视化   | 逻辑节点                  | 程序化生成节点                 |

常见组合是：

```text
PCG 生成场景点位和物体
Blueprint 负责这些物体的交互逻辑
```

例如：

```text
PCG 生成敌方巡逻点
蓝图 AI 使用这些点进行巡逻
```

------

## 8. PCG 和 World Partition 的关系

PCG 很适合与 World Partition 一起使用。

World Partition 负责：

```text
大地图按 Cell 流式加载
```

PCG 负责：

```text
每个区域内按规则生成内容
```

例如开放世界中：

```text
玩家靠近某个区域
↓
World Partition 加载该区域 Cell
↓
PCG 根据规则生成树、石头、道路设施、敌方营地
↓
玩家远离后区域卸载
```

需要注意的是：

> 大型开放世界项目中，PCG 生成内容最好考虑缓存、烘焙或分区生成，不要让所有复杂 PCG 都在运行时实时计算。

------

## 9. PCG 和 Nanite / HLOD 的关系

### 9.1 PCG + Nanite

PCG 可以大量生成 Nanite 静态网格，例如：

```text
岩石
建筑残骸
山体碎石
废墟构件
高精度扫描资产
```

Nanite 负责高面数模型的渲染效率，PCG 负责它们的规则化分布。

但注意：

> Nanite 不等于免费。数量、材质复杂度、阴影、碰撞、实例化方式仍然会影响性能。

------

### 9.2 PCG + HLOD

PCG 生成的大量远景物体，如果不处理，会给开放世界带来性能压力。

HLOD 可以把远处的一批物体合并或替换为简化版本。

典型组合：

```text
近处：
PCG 生成真实树木、岩石、建筑道具

远处：
HLOD 显示合并后的简化代理
```

在大型场景中，PCG 生成的内容应提前考虑 HLOD Layer、实例化方式和可见距离。

------

## 10. PCG 的生成模式

PCG 通常有两种使用方式：

### 10.1 编辑器生成

这是最常见、最推荐的方式。

在编辑器中点击 Generate，生成内容后保存。

适合：

```text
森林
道路设施
建筑周围杂物
山地岩石
营地布局
城市道具
```

优点是运行时压力小。

------

### 10.2 运行时生成

PCG 也可以用于运行时生成，例如：

```text
随机地牢
动态任务地点
玩家建造系统
Roguelike 地图
动态战场变化
```

官方文档说明 PCG 节点图可用于编辑器和运行时生成。([Epic Games Developers](https://dev.epicgames.com/documentation/unreal-engine/procedural-content-generation-framework-node-reference-in-unreal-engine?utm_source=chatgpt.com))

但运行时 PCG 要特别注意：

- 生成频率；
- 节点复杂度；
- 是否阻塞主线程；
- 实例数量；
- 碰撞开销；
- 网络同步；
- 存档与复现。

------

## 11. GPU PCG

UE5 的 PCG 也在逐步增强 GPU 计算能力，用于更高效地处理大量点和生成数据。

PCG 的 GPU 路径适合处理规模大的点运算，例如大面积植被、地表采样等。UE 5.6 之后，PCG 性能和运行时/编辑器执行效率有明显改进，相关讨论和文档也提到多线程执行、调度逻辑迁移出 Game Thread 等优化方向。([Reddit](https://www.reddit.com/r/UnrealProcedural/comments/1kltdxx/pcg_changes_in_unreal_engine_56/?utm_source=chatgpt.com))

但实际项目中不要一开始就依赖 GPU PCG。建议先掌握普通 PCG Graph，再根据性能瓶颈决定是否使用 GPU 节点。

------

## 12. PCG 的典型应用场景

### 12.1 森林和植被

规则示例：

```text
坡度 < 25°：生成树
坡度 25°~45°：生成灌木
坡度 > 45°：生成岩石
海拔高：减少树木，增加碎石
靠近道路：清除植被
```

------

### 12.2 道路和管线

沿 Spline 生成：

```text
路灯
护栏
电线杆
道路标识
围栏
管道支架
战壕边缘物
```

------

### 12.3 建筑和城市道具

根据建筑边界或街区规则生成：

```text
垃圾桶
长椅
广告牌
路障
车辆
空调外机
街边植被
```

------

### 12.4 战场和仿真场景

这类场景非常适合 PCG。

例如可以生成：

```text
敌方营地
雷达阵地
伪装网
沙袋
车辆掩体
巡逻路线
传感器点位
弹坑
碎石
高原植被
山地道路设施
```

对于你的三维作战仿真场景，可以把 PCG 用来快速生成大范围地形细节和目标环境。

------

## 13. 面向高原侦察仿真场景的 PCG 设计思路

你之前的场景包括高原地区、风场、仿生飞行器、侦察目标等。PCG 可以这样用：

### 13.1 地形环境生成

```text
根据海拔生成不同植被
根据坡度生成裸岩和碎石
根据水系附近生成湿地或低矮植被
根据山脊线生成裸露岩层
```

### 13.2 敌方目标区域

```text
在平坦区域生成营地
在道路附近生成补给点
在制高点生成雷达站
在山谷入口生成检查站
在隐蔽区域生成伪装目标
```

### 13.3 飞行任务辅助

PCG 不仅可以生成视觉物体，也可以生成“逻辑点”：

```text
侦察兴趣点
传感器探测点
威胁区边界点
飞行航路参考点
AI 巡逻点
可视域分析采样点
```

这对仿真系统很有价值。

------

## 14. PCG 的优点

### 14.1 大幅提高场景制作效率

尤其是开放世界、大地形、大量重复但需变化的内容。

### 14.2 规则可复用

同一个 PCG Graph 可以用于多个区域。

例如一个“高原山地植被生成器”，可以在不同地图中复用。

### 14.3 修改方便

地形、道路、建筑变化后，可以重新生成内容，不必手动全部调整。

### 14.4 容易形成工具化流程

技术美术可以把 PCG Graph 封装成工具，让关卡设计师直接使用参数。

例如：

```text
森林密度
岩石比例
随机种子
生成半径
最小道路距离
植被类型
```

------

## 15. PCG 的缺点和注意点

### 15.1 不是“自动生成好看场景”的魔法

PCG 的效果取决于规则设计、资产质量和参数调试。

规则不好会生成：

```text
树长在石头里
路灯浮空
物体互相穿插
重复感严重
性能爆炸
```

------

### 15.2 调试成本可能较高

节点图复杂后，很容易出现：

- 点不知道从哪里来；
- 某个 Filter 把点全删了；
- Attribute 名称写错；
- 生成结果不稳定；
- 种子变化导致布局变了。

建议分阶段调试，每一段都查看 Debug Points。

------

### 15.3 运行时生成要谨慎

运行时生成虽然强大，但可能带来：

- 卡顿；
- 内存峰值；
- 主线程压力；
- 网络同步问题；
- 存档问题；
- 生成结果不可控。

大型项目中，环境类 PCG 通常更推荐编辑器生成或离线构建。

------

### 15.4 资产管理很重要

PCG 会放大资产问题。

如果一个树模型、石头模型、材质或碰撞设置不好，PCG 大量生成后问题会被成倍放大。

------

## 16. 一个简单 PCG Graph 示例：山地植被生成

目标：

> 在山地地形上自动生成树、灌木和岩石。

逻辑可以是：

```text
Input: Landscape
↓
Surface Sampler
    Density = 0.2
↓
Get Surface Normal / Slope
↓
Filter by Slope
    0°~20°：保留树木点
    20°~35°：保留灌木点
    35°以上：保留岩石点
↓
Filter by Height
    高海拔减少树木
↓
Distance to Road Filter
    道路 5m 内清除大树
↓
Transform Points
    Random Yaw = 0°~360°
    Random Scale = 0.8~1.3
↓
Static Mesh Spawner
    Tree / Shrub / Rock
```

这样就能得到一个基础的高原山地环境生成器。

------

## 17. 项目中推荐的 PCG 使用原则

比较稳妥的做法是：

```text
先用 PCG 做环境批量生成
再用人工手调关键区域
最后对性能和 HLOD 做优化
```

不要完全依赖 PCG 生成所有内容。

建议分层：

```text
基础层：
地形植被、岩石、草

结构层：
道路、围栏、管线、建筑周围设施

玩法层：
敌方营地、巡逻点、任务目标

手工层：
关键叙事点、镜头区域、玩家必经路线
```

这样既能提高效率，又能保持设计质量。

------

## 18. 一句话总结

**PCG 是 UE5 的程序化内容生成框架，它通过节点图把“手工摆放场景物体”变成“定义生成规则”，特别适合开放世界、山地、森林、城市、战场和仿真场景；它能极大提高场景制作效率，但需要良好的规则设计、资产管理和性能优化，不能简单理解为一键生成高质量世界。**