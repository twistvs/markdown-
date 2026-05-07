下面介绍 **Unreal Engine 5 的 World Partition**。

## 1. World Partition 是什么？

**World Partition 是 UE5 的大型世界管理与流式加载系统**。

它的核心作用是：

> 把一个巨大的关卡自动切分成很多网格 Cell，然后根据玩家、摄像机或指定 Streaming Source 的位置，自动加载附近区域、卸载远处区域。

Epic 官方文档将 World Partition 描述为一种自动化数据管理和距离式流送系统，用来替代 UE4 时代需要手动管理的 Level Streaming。([Epic Games Developers](https://dev.epicgames.com/documentation/unreal-engine/world-partition-in-unreal-engine?utm_source=chatgpt.com))

简单理解：

```text
传统方式：
多个子关卡 + 手动设置加载/卸载逻辑

World Partition：
一个大世界 + 自动网格切分 + 自动流式加载
```

------

## 2. 它解决了什么问题？

在 UE4 或传统大地图制作中，开放世界通常需要：

- 手动拆分 Persistent Level 和 Sub Level；
- 手动设置 Level Streaming Volume；
- 手动管理每个子关卡什么时候加载；
- 多人协作时容易出现 `.umap` 冲突；
- 大地图编辑时加载慢、保存慢、协作困难。

World Partition 的目标是让大世界开发更自动化。

它可以：

- 自动按空间位置切分地图；
- 根据玩家位置自动加载区域；
- 编辑器中只加载你需要编辑的区域；
- 支持多人协作；
- 与 HLOD、Data Layer、One File Per Actor 配合使用；
- 更适合开放世界、城市、战场、仿真场景等大型地图。

------

## 3. World Partition 的基本工作方式

### 3.1 一个 Persistent Level

World Partition 通常使用一个主关卡，而不是像 UE4 那样拆很多子关卡。

所有 Actor 都可以放在这个大世界里。

### 3.2 自动网格切分

系统会把世界划分成很多 Cell。

可以把它想象成这样：

```text
+------+------+------+------+
| Cell | Cell | Cell | Cell |
+------+------+------+------+
| Cell | 玩家 | Cell | Cell |
+------+------+------+------+
| Cell | Cell | Cell | Cell |
+------+------+------+------+
```

玩家附近的 Cell 被加载，远处的 Cell 被卸载。

### 3.3 Streaming Source

World Partition 会根据 Streaming Source 判断加载范围。

最常见的 Streaming Source 是：

- Player；
- Camera；
- Player Controller；
- 自定义 Streaming Source Component。

例如开放世界游戏中，玩家走到某个区域，周围 Cell 自动加载；玩家远离后，之前区域自动卸载。

### 3.4 Runtime Grid

Runtime Grid 决定 World Partition 在运行时如何切分和加载。

常见参数包括：

```text
Cell Size：每个网格单元大小
Loading Range：加载半径
```

Cell Size 越小，流送更细，但管理开销更大。

Cell Size 越大，流送更粗，可能一次加载太多内容。

------

## 4. World Partition 和 Level Streaming 的区别

| 对比项   | 传统 Level Streaming | World Partition          |
| -------- | -------------------- | ------------------------ |
| 地图组织 | 多个子关卡           | 一个大关卡自动分区       |
| 加载方式 | 手动配置             | 自动距离流送             |
| 协作     | 容易发生关卡文件冲突 | 更适合多人协作           |
| 编辑方式 | 打开/关闭子关卡      | 选择区域加载             |
| 适合项目 | 中小型地图、线性关卡 | 开放世界、大型场景       |
| 控制精度 | 手动控制强           | 自动化程度高             |
| 学习成本 | 传统流程成熟         | 新系统，需要理解配套机制 |

------

## 5. 和 One File Per Actor 的关系

World Partition 通常会配合 **One File Per Actor，简称 OFPA**。

传统关卡里，很多 Actor 都保存在同一个 `.umap` 文件中。多人协作时，如果两个人同时改同一个关卡，很容易冲突。

OFPA 的思路是：

> 每个 Actor 单独保存成一个外部文件。

这样带来的好处是：

- A 修改一栋建筑；
- B 修改一盏灯；
- C 修改一辆车；

只要他们改的是不同 Actor，就不容易产生同一个关卡文件冲突。

Epic 官方文档也说明，One File Per Actor 允许把 Actor 保存为独立文件，从而减少多人协作中的文件冲突。([Epic Games Developers](https://dev.epicgames.com/documentation/unreal-engine/one-file-per-actor-in-unreal-engine?utm_source=chatgpt.com))

------

## 6. 和 Data Layer 的关系

**Data Layer** 用来控制一组 Actor 是否加载、显示或参与运行时逻辑。

它更像是“逻辑分组”，而不是空间切分。

例如你可以做：

```text
Data Layer: 白天版本
Data Layer: 夜晚版本
Data Layer: 任务前状态
Data Layer: 任务后状态
Data Layer: 战斗破坏前
Data Layer: 战斗破坏后
```

World Partition 负责：

```text
按空间距离加载地图
```

Data Layer 负责：

```text
按逻辑状态加载内容
```

两者经常一起用。

例如一个城市开放世界：

- World Partition 控制玩家附近城区加载；
- Data Layer 控制某片区域是“和平状态”还是“战斗后废墟状态”。

------

## 7. 和 HLOD 的关系

**HLOD，Hierarchical Level of Detail，层级细节模型**，用于远距离显示。

在大型地图中，如果远处所有建筑、树木、道路都保持原始模型，会非常消耗性能。

HLOD 的做法是：

> 把远处一组物体合并或简化成低成本代理模型。

例如：

```text
近处：
每栋楼、每扇窗、每个空调外机都是独立模型

远处：
整片街区变成一个简化后的 HLOD 模型
```

World Partition 决定区域是否加载。

HLOD 决定远处区域以什么简化形式显示。

二者结合后，开放世界可以实现：

- 近处加载真实 Actor；
- 中远距离显示简化模型；
- 非必要区域卸载。

------

## 8. World Partition 的典型工作流

### 8.1 创建支持 World Partition 的关卡

在 UE5 中，新建 Open World 类型关卡通常会默认启用 World Partition。

也可以对已有关卡进行转换。

常见入口类似：

```text
Tools → Convert Level
```

或者：

```text
Content Browser → 右键 Level → Add World Partition Streaming Support
```

不同 UE5 版本菜单名称可能略有变化。

------

### 8.2 打开 World Partition Editor

路径通常是：

```text
Window → World Partition → World Partition Editor
```

在这个窗口中，你可以看到整个世界被切成网格。

你可以选择某个区域：

```text
Load Region
Unload Region
```

这样编辑器不必一次性加载整个大地图。

------

### 8.3 设置 Actor 的 World Partition 属性

每个 Actor 通常会有相关设置，例如：

```text
Is Spatially Loaded
Runtime Grid
Data Layers
HLOD Layer
```

其中最重要的是 **Is Spatially Loaded**。

#### Is Spatially Loaded = true

Actor 会根据玩家距离加载/卸载。

适合：

- 建筑；
- 石头；
- 树木；
- 道路；
- 场景道具；
- 普通敌人刷新点。

#### Is Spatially Loaded = false

Actor 不受空间距离控制，通常一直加载，或者由其他系统控制。

适合：

- GameMode 相关管理器；
- 全局天气控制器；
- 任务系统管理器；
- 全局音频管理器；
- 需要常驻的逻辑 Actor。

------

## 9. World Partition 的优点

### 9.1 适合大型开放世界

World Partition 最适合：

- 开放世界游戏；
- 大型城市；
- 大型战场；
- 数字孪生；
- 地理仿真；
- 大规模建筑群；
- 大地图训练场景。

你之前做三维作战场景、高原地区仿真、飞行器侦察场景，这类项目就很适合考虑 World Partition。

------

### 9.2 编辑器性能更好

大地图不必一次性全部加载。

可以只加载正在编辑的区域，减少：

- 内存占用；
- 编辑器卡顿；
- 打开关卡时间；
- 保存时间。

------

### 9.3 更适合多人协作

配合 One File Per Actor 后，团队成员可以同时编辑同一个大世界中的不同 Actor，冲突比传统单 `.umap` 工作流少很多。([Epic Games Developers](https://dev.epicgames.com/documentation/unreal-engine/one-file-per-actor-in-unreal-engine?utm_source=chatgpt.com))

------

### 9.4 自动化程度高

不需要像 UE4 那样手动维护大量 Sub Level 和 Streaming Volume。

对于开放世界项目，World Partition 可以显著降低地图组织成本。

------

## 10. World Partition 的缺点和注意点

### 10.1 学习成本较高

World Partition 本身不难，但它通常和这些系统一起出现：

```text
Data Layer
HLOD
One File Per Actor
Runtime Grid
Streaming Source
Landscape
PCG
World Partition Builder
```

真正做项目时，需要整体理解。

------

### 10.2 不适合所有项目

如果你的项目是：

- 小型室内关卡；
- 线性剧情关卡；
- 房间制地图；
- 移动端轻量项目；
- 完全手动控制加载的项目；

World Partition 不一定是最佳选择。

传统 Level Streaming 或普通关卡可能更简单。

------

### 10.3 Actor 引用要小心

World Partition 下，不同 Cell 的 Actor 可能不会同时加载。

因此跨区域直接引用 Actor 可能导致问题。

例如：

```text
A 区域的机关直接引用 B 区域的门
```

如果 B 区域还没加载，这个引用就可能失效或产生加载依赖。

更推荐用：

- 软引用 Soft Object Reference；
- Event Dispatcher；
- Gameplay Tag；
- Subsystem；
- Interface；
- Data Asset；
- 任务系统中转。

------

### 10.4 Level Blueprint 不宜承载大量逻辑

在 World Partition 项目中，尽量不要把大量关卡逻辑写在 Level Blueprint 里。

官方论坛中也有开发者提到，Blueprint Classes 通常比 Level Blueprint 更适合 World Partition 世界，因为 Level Blueprint 引用的 Actor 可能会被标记为 Always Loaded。([Epic Developer Community Forums](https://forums.unrealengine.com/t/world-partition-world-streaming-documentation-questions/726488?utm_source=chatgpt.com))

更推荐：

```text
Actor Blueprint
World Subsystem
Game Instance Subsystem
Data Asset
Manager Actor
Gameplay Ability / Component
```

------

### 10.5 HLOD 构建需要时间

大型地图的 HLOD 生成可能很耗时，而且设置不合理会导致：

- 远景穿帮；
- 材质错误；
- 模型合并效果差；
- 构建时间过长；
- 包体增大；
- 运行时内存占用异常。

------

## 11. 常见关键参数解释

### Cell Size

每个分区网格的大小。

```text
Cell Size 小：
加载更精细，但 Cell 数量多，管理成本高

Cell Size 大：
管理简单，但一次加载内容多，峰值内存高
```

开放世界中通常需要结合场景密度调试。

------

### Loading Range

玩家或 Streaming Source 周围多远范围内的 Cell 会被加载。

```text
Loading Range 太小：
可能出现加载不及时、远处突然弹出

Loading Range 太大：
加载太多内容，性能和内存压力变大
```

------

### Runtime Grid

一个世界可以有不同 Runtime Grid。

例如：

```text
Grid_Environment：大地形、大建筑
Grid_SmallProps：小道具
Grid_Gameplay：玩法相关 Actor
```

但不要滥用 Runtime Grid。网格越复杂，调试成本越高。

------

### Is Spatially Loaded

决定 Actor 是否跟随空间流送。

```text
true：根据距离加载/卸载
false：不按空间距离卸载
```

很多 World Partition 问题都和这个设置有关。

------

## 12. 对大型三维仿真/作战场景的建议

如果你的项目是高原地区侦察仿真、无人机/仿生飞行器飞行场景，可以这样组织：

```text
World Partition：
管理大范围地形、山体、道路、建筑、植被

HLOD：
管理远处山体、村落、雷达站、建筑群的简化显示

Data Layer：
管理不同任务态势
例如：和平状态、敌方部署状态、打击后状态、夜间状态

One File Per Actor：
方便团队协作编辑大量目标、传感器、建筑、载具

Streaming Source：
可以绑定到玩家、摄像机、飞行器、无人机视角
```

特别注意飞行器项目：

> 飞行速度越快，World Partition 的 Loading Range 通常要越大，否则高速飞行时容易出现前方区域加载不及时。

如果是高空侦察，还要重点调：

- HLOD 可视距离；
- Landscape LOD；
- Nanite 远景表现；
- Streaming Source 加载半径；
- World Origin / 大世界坐标精度；
- 地形材质远距离简化。

------

## 13. 一句话总结

**World Partition 是 UE5 用来管理大型开放世界的自动分区和流式加载系统，它把大地图拆成网格 Cell，并根据玩家或 Streaming Source 的位置自动加载/卸载区域；它适合开放世界、城市、战场和大规模仿真场景，但需要配合 HLOD、Data Layer、One File Per Actor 等系统一起设计，不能简单当成“自动优化按钮”。**