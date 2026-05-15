下面回答 **七、AI相关问题**。这一部分在 UE5 实习面试中也很常见，尤其是如果你的项目里做过敌人巡逻、追击、攻击、感知玩家，面试官很可能会问到：

- UE AI 常用系统有哪些？
- AIController 是什么？
- Behavior Tree 行为树是什么？
- Blackboard 黑板是什么？
- NavMesh 是什么？
- Move To 为什么不动？
- AI Perception 是什么？
- 行为树中 Task、Decorator、Service 分别是什么？

------

# 七、AI相关问题

## 1. UE AI 常用系统有哪些？

UE 中常见的 AI 系统主要包括：

```text
AIController
Behavior Tree
Blackboard
NavMesh
AI Perception
Pawn Sensing
EQS
BTTask
BTService
BTDecorator
```

实习岗最常问的通常是前五个。

可以先这样回答：

> UE 中 AI 通常由 AIController 控制 Pawn，通过 Behavior Tree 组织行为逻辑，通过 Blackboard 存储决策数据，通过 NavMesh 实现寻路移动，通过 AI Perception 或 Pawn Sensing 感知玩家。

------

# 2. AIController 是什么？

## 2.1 基本概念

`AIController` 是 AI 的控制器。

在 UE 中，角色本身通常是 `Pawn` 或 `Character`，而真正做“决策”的对象是 `Controller`。

玩家角色通常由：

```text
PlayerController 控制
```

AI 敌人通常由：

```text
AIController 控制
```

例如：

```text
BP_EnemyCharacter
    被 BP_EnemyAIController 控制
```

------

## 2.2 AIController 负责什么？

AIController 通常负责：

```text
运行行为树
设置黑板值
控制 AI 移动
处理感知结果
管理 AI 状态
发出攻击或巡逻命令
```

比如敌人发现玩家后：

```text
AI Perception 感知到玩家
    ↓
AIController 设置黑板 TargetActor = Player
    ↓
Behavior Tree 根据 TargetActor 执行追击或攻击
```

------

## 2.3 Character 和 AIController 的分工

一般可以这样分：

| 类             | 职责                             |
| -------------- | -------------------------------- |
| EnemyCharacter | 血量、动画、攻击表现、受击、死亡 |
| AIController   | 感知、决策、运行行为树、设置黑板 |
| Behavior Tree  | 行为流程                         |
| Blackboard     | 存储行为数据                     |

------

## 2.4 面试推荐回答

> AIController 是 AI Pawn 的控制器，类似玩家的 PlayerController。AIController 负责运行行为树、更新黑板数据、处理感知结果和控制 AI 移动。敌人角色本身负责动画、血量、攻击等表现和能力，而 AIController 更偏向决策和控制。

------

# 3. Behavior Tree 行为树是什么？

## 3.1 基本概念

`Behavior Tree`，行为树，是 UE 中组织 AI 决策逻辑的工具。

它用树状结构描述 AI 应该做什么。

例如一个敌人 AI：

```text
是否看到玩家？
    是 → 追击玩家
        距离足够近 → 攻击
    否 → 巡逻
```

对应行为树可以理解为：

```text
Root
 └── Selector
      ├── Sequence：攻击玩家
      │    ├── 判断 TargetActor 是否存在
      │    ├── 判断距离是否足够近
      │    └── Attack
      ├── Sequence：追击玩家
      │    ├── 判断 TargetActor 是否存在
      │    └── Move To TargetActor
      └── Sequence：巡逻
           ├── 获取巡逻点
           └── Move To PatrolPoint
```

------

## 3.2 行为树有什么好处？

行为树的优点：

```text
逻辑清晰
可视化
易于调试
适合复杂 AI 行为
可以拆分 Task、Service、Decorator
设计师也能调整部分逻辑
```

如果不用行为树，可能会在 Tick 中写大量 if-else：

```cpp
if (CanSeePlayer)
{
    if (InAttackRange)
    {
        Attack();
    }
    else
    {
        Chase();
    }
}
else
{
    Patrol();
}
```

行为树可以把这些逻辑可视化，维护起来更清晰。

------

## 3.3 面试推荐回答

> Behavior Tree 是 UE 中用于组织 AI 决策逻辑的树状结构。它可以通过 Selector、Sequence、Task、Decorator、Service 等节点描述 AI 行为，比如巡逻、发现玩家、追击、攻击。相比在 Tick 中写大量 if-else，行为树更清晰、更易调试，也更适合复杂 AI。

------

# 4. Blackboard 黑板是什么？

## 4.1 基本概念

`Blackboard` 是行为树使用的数据存储区。

行为树负责“怎么决策”，黑板负责“存数据”。

常见黑板变量：

```text
TargetActor：当前目标
PatrolLocation：巡逻点位置
CanSeePlayer：是否看到玩家
DistanceToTarget：距离目标多远
IsAttacking：是否正在攻击
LastKnownLocation：最后看到玩家的位置
```

------

## 4.2 行为树和黑板的关系

可以这样理解：

```text
Blackboard：存储 AI 当前信息
Behavior Tree：根据这些信息决定行为
```

例如：

```text
Blackboard.TargetActor != None
    → 行为树执行追击或攻击

Blackboard.TargetActor == None
    → 行为树执行巡逻
```

------

## 4.3 黑板变量类型

常见类型：

```text
Object
Vector
Bool
Float
Int
Enum
Name
String
Class
```

例如：

```text
TargetActor：Object 类型，Base Class = Actor
PatrolLocation：Vector 类型
bCanSeePlayer：Bool 类型
AttackRange：Float 类型
```

------

## 4.4 面试推荐回答

> Blackboard 是行为树的数据存储区，用来保存 AI 决策需要的数据，比如目标 Actor、巡逻点、是否看到玩家、最后已知位置等。Behavior Tree 会根据 Blackboard 中的数据选择执行不同分支。比如 TargetActor 不为空时追击玩家，TargetActor 为空时执行巡逻。

------

# 5. Behavior Tree 和 Blackboard 的关系

这是非常高频的问题。

可以直接这样回答：

> Behavior Tree 负责行为逻辑，Blackboard 负责存储行为数据。行为树中的 Decorator、Task、Service 都可以读取或修改黑板变量。比如 AIController 感知到玩家后，把玩家写入黑板的 TargetActor，行为树检测到 TargetActor 不为空，就执行追击或攻击分支。

举例：

```text
AI Perception 感知到玩家
    ↓
AIController 设置 Blackboard.TargetActor = Player
    ↓
Behavior Tree 的 Decorator 判断 TargetActor Is Set
    ↓
执行 Move To TargetActor
```

再比如丢失玩家：

```text
AI Perception 丢失玩家
    ↓
设置 Blackboard.TargetActor = None
    ↓
行为树追击分支失败
    ↓
切换到巡逻分支
```

------

# 6. 行为树中的 Selector 和 Sequence 是什么？

## 6.1 Selector

`Selector` 可以理解为“选择器”。

它会从左到右尝试执行子节点，只要有一个成功，就认为成功。

常用于：

```text
优先级选择
```

例如：

```text
Selector
 ├── 攻击玩家
 ├── 追击玩家
 └── 巡逻
```

含义：

```text
能攻击就攻击
不能攻击就追击
不能追击就巡逻
```

------

## 6.2 Sequence

`Sequence` 可以理解为“顺序执行”。

它会从左到右执行子节点，只要有一个失败，整个 Sequence 就失败。

常用于：

```text
一组必须全部满足的行为
```

例如：

```text
Sequence：攻击玩家
 ├── 判断 TargetActor 是否存在
 ├── 判断距离是否小于攻击范围
 └── 执行攻击
```

含义：

```text
必须有目标
并且距离足够近
才能攻击
```

------

## 6.3 对比总结

| 节点     | 逻辑           | 类似 |
| -------- | -------------- | ---- |
| Selector | 一个成功就成功 | OR   |
| Sequence | 一个失败就失败 | AND  |

------

## 6.4 面试推荐回答

> Selector 是选择器，会从左到右选择第一个可以成功执行的分支，常用于 AI 行为优先级，比如攻击、追击、巡逻。Sequence 是顺序节点，要求所有子节点按顺序成功，常用于一组条件和行为，比如有目标、距离足够近，然后执行攻击。

------

# 7. Task、Decorator、Service 分别是什么？

这是行为树面试重点。

------

## 7.1 Task 是什么？

`Task` 是行为树中真正执行动作的节点。

例如：

```text
Move To
Wait
Rotate To Face BB Entry
Play Animation
Attack
Find Random Patrol Point
```

自定义 Task 可以实现：

```text
攻击玩家
选择巡逻点
释放技能
播放动画
呼叫支援
```

简单说：

> Task 是“做事”的节点。

------

## 7.2 Decorator 是什么？

`Decorator` 是条件判断节点。

它通常挂在某个分支上，用来判断这个分支能不能执行。

例如：

```text
Blackboard TargetActor Is Set
DistanceToTarget < AttackRange
Health < 30%
CanSeePlayer == true
```

简单说：

> Decorator 是“能不能做”的判断。

------

## 7.3 Service 是什么？

`Service` 是挂在 Composite 节点上的周期性更新逻辑。

它通常用于定期更新黑板数据。

例如：

```text
每 0.2 秒更新玩家距离
每 0.5 秒检测是否能看到玩家
定期选择最近敌人
定期更新是否在攻击范围内
```

简单说：

> Service 是“持续更新信息”的节点。

------

## 7.4 对比总结

| 节点      | 作用         | 例子                       |
| --------- | ------------ | -------------------------- |
| Task      | 执行动作     | MoveTo、Attack、Wait       |
| Decorator | 判断条件     | 是否有目标、是否在攻击范围 |
| Service   | 周期更新数据 | 更新距离、更新目标         |

------

## 7.5 面试推荐回答

> Task 是行为树中真正执行动作的节点，比如 Move To、Wait、Attack；Decorator 是条件判断节点，比如目标是否存在、距离是否足够近；Service 是周期性更新节点，常用于更新黑板数据，比如目标距离、是否能攻击等。

------

# 8. NavMesh 是什么？

## 8.1 基本概念

`NavMesh` 是导航网格，用来告诉 AI 哪些地方可以走。

UE 中常用：

```text
NavMeshBoundsVolume
```

把它放到关卡中并覆盖 AI 可以移动的区域，UE 会生成一片绿色导航区域。

AI 的 `Move To` 通常依赖 NavMesh。

------

## 8.2 NavMesh 有什么用？

它用于：

```text
AI 寻路
Move To 目标点
绕开障碍物
计算可行走区域
随机巡逻点
```

比如敌人追击玩家：

```text
AI Move To Player
    ↓
在 NavMesh 上计算路径
    ↓
沿路径移动到玩家附近
```

------

## 8.3 如果没有 NavMesh 会怎样？

常见结果：

```text
AI Move To 失败
AI 站着不动
行为树 Move To 一直失败
```

------

## 8.4 面试推荐回答

> NavMesh 是导航网格，用来表示 AI 可以行走的区域。UE 中通常通过 NavMeshBoundsVolume 生成导航区域，AI 的 Move To 会基于 NavMesh 计算路径。如果没有正确生成 NavMesh，AI 可能无法移动或 Move To 失败。

------

# 9. Move To 节点是什么？

`Move To` 是行为树中常见 Task，用于让 AI 移动到目标位置或目标 Actor。

它可以接收黑板中的：

```text
Vector 类型：移动到某个坐标
Object 类型：移动到某个 Actor
```

例如：

```text
Move To TargetActor
Move To PatrolLocation
```

------

## 9.1 Move To 使用 Actor 目标

黑板 Key：

```text
TargetActor
类型：Object
Base Class：Actor
```

行为树中：

```text
Move To TargetActor
```

AI 会移动到这个 Actor 附近。

------

## 9.2 Move To 使用 Vector 目标

黑板 Key：

```text
PatrolLocation
类型：Vector
```

行为树中：

```text
Move To PatrolLocation
```

AI 会移动到这个位置。

------

## 9.3 Acceptance Radius

`Acceptance Radius` 是接受半径。

意思是：

> AI 不需要精确移动到目标点，只要距离目标小于这个半径，就认为 Move To 成功。

比如：

```text
Acceptance Radius = 100
```

AI 距离目标 100 单位以内就算到达。

------

# 10. 黑板中有 Player，为什么 Move To 中无法选择 Player？

这个你之前也遇到过，是非常典型的 UE AI 问题。

常见原因如下。

------

## 10.1 黑板变量类型不对

Move To 通常只能选择：

```text
Object 类型
Vector 类型
```

如果你的 `Player` 是 Bool、String、Class 或其他类型，就不能被 Move To 选择。

正确设置：

```text
Key Name：Player
Key Type：Object
Base Class：Actor 或 Pawn 或 Character
```

------

## 10.2 Object 类型没有设置 Base Class

如果黑板 Key 是 Object 类型，但 Base Class 没有设置，或者设置得不合适，Move To 可能无法识别。

推荐：

```text
Base Class = Actor
```

或者更具体：

```text
Base Class = Character
```

------

## 10.3 行为树没有使用正确 Blackboard

Behavior Tree 右上角需要绑定正确的 Blackboard。

如果行为树使用的不是你编辑的那个黑板，那么 Move To 里自然找不到对应 Key。

------

## 10.4 没有保存或刷新编辑器

有时你添加了黑板 Key，但行为树没有刷新。

可以尝试：

```text
保存 Blackboard
保存 Behavior Tree
重新打开 Behavior Tree
```

------

## 10.5 面试推荐回答

> Move To 只能使用 Vector 类型或 Object 类型的黑板 Key。如果我的黑板中 Player 无法在 Move To 中选择，我会检查 Player 是否是 Object 类型，并且 Base Class 是否设置为 Actor、Pawn 或 Character；还要确认行为树绑定的是正确的 Blackboard，并保存刷新资源。

------

# 11. AI Move To 不动，怎么排查？

这是非常常见的排查题。

可以按以下顺序回答。

------

## 11.1 是否有 NavMesh

按 `P` 可以显示导航区域。

如果没有绿色区域，说明 NavMesh 没生成。

检查：

```text
是否放置 NavMeshBoundsVolume
是否覆盖 AI 和目标区域
地面是否可导航
Runtime Generation 是否正确
```

------

## 11.2 AI 是否有 AIController

检查敌人角色：

```text
AI Controller Class 是否设置正确
Auto Possess AI 是否设置为 Placed in World or Spawned
```

如果没有 Controller，AI 不会执行行为树，也不能 Move To。

------

## 11.3 是否运行了 Behavior Tree

AIController 中需要：

```blueprint
Run Behavior Tree
```

或 C++：

```cpp
RunBehaviorTree(BehaviorTreeAsset);
```

------

## 11.4 Move To 的目标是否有效

如果目标 Actor 是空：

```text
TargetActor = None
```

Move To 会失败。

如果目标 Vector 是默认值：

```text
(0, 0, 0)
```

AI 可能跑向错误位置。

------

## 11.5 角色移动组件是否正常

如果是 Character，通常需要：

```text
CharacterMovementComponent
```

检查：

```text
Movement Mode 是否正常
Max Walk Speed 是否大于 0
是否被禁用移动
是否被碰撞卡住
```

------

## 11.6 碰撞是否阻挡

AI 可能被墙体、胶囊体、其他 Pawn 卡住。

检查：

```text
Capsule 碰撞
Nav Agent Radius / Height
障碍物碰撞
是否和其他 AI 互相阻挡
```

------

## 11.7 面试推荐回答

> AI Move To 不动时，我会先按 P 查看是否有 NavMesh，然后检查 AI 是否被正确的 AIController Possess，Auto Possess AI 是否设置正确，AIController 是否 Run Behavior Tree。接着检查 Move To 的目标黑板值是否有效，角色移动组件是否启用，Max Walk Speed 是否正常，以及碰撞是否把 AI 卡住。

------

# 12. AI Perception 是什么？

## 12.1 基本概念

`AI Perception` 是 UE 的 AI 感知系统。

它可以让 AI 感知：

```text
视觉 Sight
听觉 Hearing
伤害 Damage
预测 Prediction
触碰 Touch
团队 Team
```

最常用的是：

```text
AI Sight
```

也就是视觉感知。

------

## 12.2 AI Sight 常见参数

常见参数：

```text
Sight Radius：视觉半径
Lose Sight Radius：丢失视觉半径
Peripheral Vision Angle：视野角度
Max Age：感知信息保留时间
Detection by Affiliation：检测敌人/友方/中立
```

例如：

```text
Sight Radius = 1500
Lose Sight Radius = 1800
Peripheral Vision Angle = 60
```

表示 AI 在 1500 范围内、视野角内可以看到目标，目标离开到 1800 之后才完全丢失。

------

## 12.3 感知到玩家后怎么做？

通常在 AIController 中添加：

```text
AIPerceptionComponent
```

绑定事件：

```text
OnTargetPerceptionUpdated
```

当感知到玩家：

```text
Stimulus Successfully Sensed == true
    → Blackboard.TargetActor = Player
```

当丢失玩家：

```text
Stimulus Successfully Sensed == false
    → Blackboard.LastKnownLocation = Player Location
    → Blackboard.TargetActor = None
```

------

## 12.4 面试推荐回答

> AI Perception 是 UE 的 AI 感知系统，可以处理视觉、听觉、伤害等感知。常用的是 Sight，AI 感知到玩家后，可以在 AIController 中把玩家写入 Blackboard 的 TargetActor，行为树再根据 TargetActor 执行追击或攻击；丢失玩家时可以清空目标或记录最后已知位置。

------

# 13. Pawn Sensing 和 AI Perception 的区别

## 13.1 Pawn Sensing

`Pawn Sensing` 是较早的感知组件，常用于简单视觉和听觉检测。

它可以：

```text
看到 Pawn
听到声音
```

优点：

```text
简单易用
适合小 Demo
```

缺点：

```text
功能较老
扩展性不如 AI Perception
```

------

## 13.2 AI Perception

`AI Perception` 更完整、更现代，支持多种感知类型和配置。

适合：

```text
正式项目
复杂 AI
多感知类型
团队关系判断
```

------

## 13.3 面试推荐回答

> Pawn Sensing 是比较早的简单感知组件，适合快速实现看到玩家或听到声音。AI Perception 是更完整的感知系统，支持 Sight、Hearing、Damage 等多种感知类型，扩展性更强，正式项目中更推荐使用 AI Perception。

------

# 14. EQS 是什么？

## 14.1 基本概念

`EQS` 全称 Environment Query System，环境查询系统。

它用于让 AI 在环境中选择一个合适的位置或目标。

例如：

```text
找一个离玩家近但有掩体的位置
找一个适合攻击玩家的位置
找一个远离危险区域的位置
找一个随机巡逻点
找一个视野能看到玩家的位置
```

------

## 14.2 EQS 示例：寻找掩体

AI 被攻击时，可以用 EQS 查询：

```text
生成周围一圈候选点
    ↓
过滤不可到达点
    ↓
过滤暴露在玩家视线中的点
    ↓
选择距离适中的点
    ↓
Move To 该点
```

------

## 14.3 面试推荐回答

> EQS 是环境查询系统，用来帮助 AI 在场景中选择合适的位置或目标，比如找掩体、找攻击位置、找巡逻点。它会生成一组候选点，然后通过过滤和评分选出最合适的结果。复杂 AI 中可以用 EQS 提高决策质量。

------

# 15. 实战问题：如何实现一个基础敌人 AI？

这是面试非常高频的问题。

可以按这个流程回答：

```text
1. 创建 EnemyCharacter
2. 创建 EnemyAIController
3. 创建 Blackboard
4. 创建 Behavior Tree
5. 在关卡中添加 NavMeshBoundsVolume
6. AIController BeginPlay 中 Run Behavior Tree
7. AI Perception 感知玩家
8. 感知到玩家后设置 Blackboard.TargetActor
9. 行为树根据 TargetActor 决定巡逻、追击、攻击
```

------

## 15.1 行为逻辑示例

```text
Root
 └── Selector
      ├── Sequence：攻击
      │    ├── TargetActor Is Set
      │    ├── InAttackRange == true
      │    └── Attack Task
      ├── Sequence：追击
      │    ├── TargetActor Is Set
      │    └── Move To TargetActor
      └── Sequence：巡逻
           ├── Find Random Patrol Location
           ├── Move To PatrolLocation
           └── Wait
```

------

## 15.2 面试推荐回答

> 我会创建 EnemyCharacter 和 EnemyAIController，用 AIController 运行 Behavior Tree。Blackboard 中保存 TargetActor、PatrolLocation、InAttackRange 等数据。AI Perception 负责感知玩家，感知到玩家后把 TargetActor 写入黑板。行为树用 Selector 做优先级选择：如果目标存在且在攻击范围内就攻击；如果目标存在但距离较远就追击；如果没有目标就巡逻。同时关卡中需要 NavMeshBoundsVolume 支持 Move To 寻路。

------

# 16. 实战问题：如何实现 AI 巡逻？

常见方案有两种。

------

## 16.1 固定巡逻点

在敌人身上配置：

```cpp
UPROPERTY(EditInstanceOnly)
TArray<AActor*> PatrolPoints;
```

行为树中：

```text
Get Next Patrol Point
    ↓
设置 Blackboard.PatrolLocation
    ↓
Move To PatrolLocation
    ↓
Wait
    ↓
选择下一个点
```

适合：

```text
守卫巡逻
固定路线敌人
关卡脚本型 AI
```

------

## 16.2 随机巡逻点

使用：

```text
Get Random Reachable Point in Radius
```

从当前位置或出生点附近找一个 NavMesh 上可到达的随机点。

流程：

```text
Find Random Location
    ↓
Set Blackboard.PatrolLocation
    ↓
Move To
    ↓
Wait
    ↓
重复
```

适合：

```text
野怪游荡
普通敌人随机移动
```

------

## 16.3 面试推荐回答

> AI 巡逻可以用固定巡逻点，也可以用随机巡逻点。固定巡逻点适合守卫路线，可以在敌人身上配置 PatrolPoints 数组，行为树依次 Move To。随机巡逻可以在 NavMesh 上用 Get Random Reachable Point in Radius 获取一个可达位置，然后写入 Blackboard 的 PatrolLocation，再让 Move To 执行。

------

# 17. 实战问题：如何实现 AI 追击和攻击玩家？

可以这样回答：

```text
AI Perception 感知玩家
    ↓
Blackboard.TargetActor = Player
    ↓
Behavior Tree 进入追击分支
    ↓
Move To TargetActor
    ↓
Service 定期计算距离
    ↓
距离小于 AttackRange
    ↓
进入攻击分支
    ↓
停止移动，朝向玩家，播放攻击 Montage
```

------

## 17.1 攻击时要注意什么？

攻击通常不是简单 Tick 扣血，而是：

```text
播放攻击 Montage
Anim Notify 控制伤害时机
使用碰撞或 Trace 检测
避免重复命中
攻击结束后恢复行为树
```

如果是 AI 近战攻击，可以：

```text
BTTask_Attack
    → 调用 EnemyCharacter.Attack()
    → 播放 Montage
    → 等待 Montage 结束
    → Finish Execute
```

------

## 17.2 面试推荐回答

> AI 追击玩家可以通过 AI Perception 设置黑板 TargetActor，行为树检测到目标后执行 Move To。可以用 Service 周期性更新距离，当距离小于攻击范围时进入攻击分支。攻击时通常停止移动并朝向玩家，播放攻击 Montage，再通过 Anim Notify 控制实际伤害判定，攻击结束后 Task 返回成功，让行为树继续决策。

------

# 18. BTTask 攻击任务如何结束？

这个问题很重要，因为很多新手写自定义 Task 时会卡住行为树。

## 18.1 立即完成型 Task

比如设置一个巡逻点：

```text
执行完设置黑板值
    ↓
Finish Execute Success
```

这种 Task 可以马上结束。

------

## 18.2 等待动画完成型 Task

攻击 Task 通常不能立即结束。

因为攻击动画需要时间。

流程：

```text
BTTask_Attack Execute
    ↓
调用角色 Attack
    ↓
播放 Montage
    ↓
绑定 Montage End
    ↓
Montage 结束时 Finish Execute
```

如果 Task 一开始就 Finish，行为树可能马上进入下一轮，导致攻击被重复触发。

------

## 18.3 面试推荐回答

> 自定义 BTTask 执行完后需要调用 Finish Execute。像设置巡逻点这种瞬时任务可以立即 Finish；攻击任务通常要等 Montage 播放结束或攻击逻辑完成后再 Finish，否则行为树会立刻继续执行，可能导致攻击重复触发或状态混乱。

------

# 19. 行为树为什么一直重复攻击？

常见原因：

```text
Attack Task 立即 Finish，下一帧又满足攻击条件
没有攻击冷却
没有 bIsAttacking 状态
Decorator 条件一直为真
Montage 没有等待结束
没有设置 Cooldown Decorator
```

解决方式：

```text
Attack Task 等待 Montage 结束再 Finish
增加攻击冷却
黑板中设置 bIsAttacking
使用 Cooldown Decorator
攻击时停止移动
```

面试回答：

> 如果行为树一直重复攻击，可能是攻击 Task 立即完成，而攻击条件一直满足。可以让攻击 Task 等待 Montage 结束后再 Finish，或者增加 Cooldown Decorator、bIsAttacking 黑板变量、攻击间隔等限制，避免每轮行为树都触发攻击。

------

# 20. AI 行为树调试方法

UE 行为树有很好的调试工具。

可以：

```text
打开 Behavior Tree
运行 PIE
选择正在运行的 AI 实例
观察当前执行节点
观察 Blackboard 值变化
查看 Move To 是否失败
查看 Decorator 是否阻止分支
```

还可以配合：

```text
按 P 显示 NavMesh
Draw Debug 显示感知范围
打印黑板变量
显示 AI MoveTo 结果
```

面试回答：

> 调试行为树时，我会在 PIE 运行时打开 Behavior Tree，选择对应 AI 实例，观察当前执行到哪个节点，以及 Blackboard 变量是否正确变化。如果 Move To 失败，我会检查 NavMesh；如果分支不执行，我会检查 Decorator 条件；如果感知无效，我会检查 AI Perception 配置和黑板赋值。

------

# 21. 第七部分综合面试模板

如果面试官问：“你了解 UE 的 AI 系统吗？”

可以这样回答：

> UE 的 AI 通常由 AIController、Behavior Tree、Blackboard、NavMesh 和 AI Perception 组成。AIController 负责控制 AI Pawn、运行行为树、处理感知结果并更新黑板。Behavior Tree 负责组织 AI 决策逻辑，Blackboard 用来存储决策数据，比如目标 Actor、巡逻点、是否在攻击范围内。NavMesh 提供可行走区域，Move To 依赖 NavMesh 进行寻路。AI Perception 用于感知玩家，比如视觉或听觉。
>
> 行为树中 Selector 通常做优先级选择，比如攻击、追击、巡逻；Sequence 表示一组条件和动作必须按顺序成功。Task 负责执行动作，Decorator 负责判断条件，Service 负责周期性更新数据。
>
> 实现敌人 AI 时，我会让 AI Perception 感知玩家并设置 Blackboard.TargetActor，行为树根据 TargetActor 判断是否追击或攻击；如果没有目标，则通过巡逻点或随机 NavMesh 点巡逻。攻击一般用 BTTask 调用角色播放 Montage，并等待 Montage 结束后 Finish Execute。

------

# 22. 简短版背诵答案

面试时可以背这个版本：

> UE AI 常用系统有 AIController、Behavior Tree、Blackboard、NavMesh 和 AI Perception。AIController 控制 AI Pawn 并运行行为树；Behavior Tree 负责决策逻辑；Blackboard 存储决策数据；NavMesh 提供寻路区域；AI Perception 负责感知玩家。
>
> 行为树中 Selector 类似优先级选择，一个分支成功就成功；Sequence 要求子节点按顺序全部成功。Task 是执行动作，比如 Move To、Attack；Decorator 是条件判断，比如目标是否存在；Service 是周期性更新，比如更新目标距离。
>
> Move To 不动时，我会检查 NavMesh、AIController、Auto Possess AI、行为树是否运行、黑板目标是否有效、CharacterMovement 是否正常。实现基础敌人 AI 时，一般是没有目标就巡逻，看到玩家就追击，进入攻击范围就播放攻击 Montage。