UE5 可以从不同层面理解“由哪些组件构成”。如果你问的是 **引擎整体架构**，可以分成这些主要部分：

```text
UE5
├─ 编辑器 Editor
├─ 运行时 Runtime
├─ 渲染系统 Rendering
├─ 物理系统 Physics
├─ 动画系统 Animation
├─ 输入系统 Input
├─ 游戏框架 Gameplay Framework
├─ 资源系统 Asset System
├─ 蓝图系统 Blueprint
├─ C++ 模块系统 Module
├─ 网络系统 Networking
├─ AI 系统 AI
├─ UI 系统 UMG / Slate
├─ 音频系统 Audio
├─ 关卡与世界系统 World / Level
├─ 反射与序列化系统 Reflection / Serialization
└─ 插件系统 Plugins
```

------

## 1. 编辑器 Editor

UE5 编辑器是你平时看到的开发界面，包括：

```text
关卡编辑器
蓝图编辑器
材质编辑器
动画编辑器
UMG 编辑器
Niagara 编辑器
行为树编辑器
控制台和输出日志
```

它主要用于创建和编辑游戏内容。

例如你拖拽模型、摆放灯光、创建蓝图、设置材质，都是在 Editor 中完成的。

------

## 2. 运行时 Runtime

Runtime 是游戏真正运行时使用的部分。

打包之后，玩家运行游戏时不会带完整编辑器，但会带运行时模块。

它负责：

```text
游戏逻辑运行
Actor 生成和销毁
Tick 更新
物理模拟
动画播放
输入响应
网络同步
渲染画面
```

------

## 3. 游戏框架 Gameplay Framework

这是 UE 里做游戏逻辑最核心的一层。

常见类包括：

```text
UObject
AActor
APawn
ACharacter
AController
APlayerController
AAIController
AGameModeBase
AGameStateBase
APlayerState
UActorComponent
USceneComponent
```

简单理解：

```text
Actor：关卡中可存在的对象
Pawn：可以被控制的 Actor
Character：带角色移动组件的 Pawn
Controller：控制 Pawn 的大脑
GameMode：游戏规则
GameState：游戏全局状态
PlayerState：玩家状态
Component：Actor 的功能组件
```

比如一个玩家角色通常是：

```text
BP_PlayerCharacter
├─ CapsuleComponent
├─ SkeletalMesh
├─ SpringArm
├─ Camera
├─ CharacterMovement
└─ AbilitySystemComponent
```

------

## 4. 组件系统 Component System

UE 中很多功能是通过组件组合出来的。

常见组件：

```text
StaticMeshComponent
SkeletalMeshComponent
CameraComponent
SpringArmComponent
BoxCollision
SphereCollision
CapsuleComponent
AudioComponent
ParticleSystemComponent
NiagaraComponent
CharacterMovementComponent
ProjectileMovementComponent
AbilitySystemComponent
```

Actor 本身更像一个容器，具体功能由组件提供。

例如一个火球 Actor 可能是：

```text
BP_Fireball
├─ SphereCollision
├─ StaticMesh / Niagara
├─ ProjectileMovement
└─ AudioComponent
```

------

## 5. 蓝图系统 Blueprint

蓝图是 UE 的可视化脚本系统。

它可以用来做：

```text
角色逻辑
UI 逻辑
关卡交互
AI 行为
动画逻辑
技能逻辑
材质参数控制
```

常见蓝图类型：

```text
Actor Blueprint
Character Blueprint
PlayerController Blueprint
Anim Blueprint
Widget Blueprint
Blueprint Interface
Blueprint Function Library
Behavior Tree Task Blueprint
Gameplay Ability Blueprint
```

蓝图本质上运行在 UE 的反射系统和虚拟机/编译系统之上，可以和 C++ 互相调用。

------

## 6. C++ 模块系统

UE5 的 C++ 工程由模块组成。

常见文件：

```text
Source/ProjectName/
├─ ProjectName.Build.cs
├─ Public/
└─ Private/
```

`Build.cs` 用来声明依赖模块，比如：

```csharp
"Core",
"CoreUObject",
"Engine",
"InputCore",
"EnhancedInput",
"GameplayAbilities",
"GameplayTags",
"GameplayTasks"
```

插件、编辑器功能、运行时功能都可以用模块组织。

------

## 7. 渲染系统 Rendering

UE5 的渲染系统负责画面输出。

主要技术包括：

```text
Nanite
Lumen
Virtual Shadow Maps
Deferred Rendering
Forward Rendering
Material System
Post Process
Global Illumination
Reflection
Anti-Aliasing
```

其中：

```text
Nanite：虚拟化几何体系统，适合高面数模型
Lumen：动态全局光照和反射系统
Virtual Shadow Maps：虚拟阴影贴图
Material：材质系统
Post Process：后处理系统
```

------

## 8. 材质系统 Material System

UE 的材质系统基于节点。

常见输入：

```text
Base Color
Metallic
Roughness
Normal
Emissive Color
Opacity
World Position Offset
```

材质可以配合：

```text
纹理 Texture
法线贴图 Normal Map
Mask
Material Instance
Parameter Collection
```

材质实例常用于运行时快速调整参数，例如改变颜色、透明度、溶解效果。

------

## 9. 动画系统 Animation

动画系统负责角色骨骼动画和动画逻辑。

常见内容：

```text
Skeleton
Skeletal Mesh
Animation Sequence
Animation Blueprint
Blend Space
Animation Montage
Anim Notify
State Machine
Control Rig
IK Rig
IK Retargeter
Pose Asset
```

你前面问过的这些都属于动画系统：

```text
Calculate Direction
Event Blueprint Update Animation
Slot
Montage
Blend Space
State Machine
Layered Blend Per Bone
```

------

## 10. 物理系统 Physics

UE5 默认使用 Chaos 物理系统。

它负责：

```text
刚体模拟
碰撞检测
Overlap
Trace
物理约束
破坏系统
布料模拟
车辆物理
```

常见相关概念：

```text
Collision Channel
Collision Preset
Object Type
Trace Channel
Block / Overlap / Ignore
Simulate Physics
Physics Material
```

------

## 11. 输入系统 Input

UE5 现在推荐使用：

```text
Enhanced Input
```

核心资源：

```text
Input Action
Input Mapping Context
Trigger
Modifier
Enhanced Input Component
Enhanced Input Local Player Subsystem
```

典型流程：

```text
创建 IA_Move
创建 IMC_Player
BeginPlay 添加 Mapping Context
在 PlayerController 或 Character 中响应 IA_Move
```

------

## 12. AI 系统

UE5 AI 系统包括：

```text
AIController
Behavior Tree
Blackboard
BTTask
BTService
BTDecorator
NavMesh
Perception
EQS
```

典型 AI 流程：

```text
AIController Possess 敌人
↓
Run Behavior Tree
↓
Blackboard 存储 Player / TargetLocation
↓
Behavior Tree 决策
↓
Move To / Attack / Patrol
```

------

## 13. UI 系统：UMG 和 Slate

UE5 UI 主要有两套：

```text
UMG：面向游戏 UI，蓝图友好
Slate：底层 C++ UI 框架
```

UMG 常见控件：

```text
Canvas Panel
Overlay
Horizontal Box
Vertical Box
TextBlock
Image
Button
ProgressBar
Border
WidgetSwitcher
ScrollBox
```

游戏里常用：

```text
HUD
血条
蓝条
背包
技能栏
菜单
对话框
伤害数字
```

------

## 14. 资源系统 Asset System

UE 的内容资源放在：

```text
Content/
```

常见资源类型：

```text
Static Mesh
Skeletal Mesh
Material
Texture
Animation
Sound
Blueprint
DataTable
Curve
Niagara System
Level
Widget Blueprint
Gameplay Effect
Gameplay Ability
```

UE 还包含：

```text
Asset Registry
Asset Manager
Soft Reference
Hard Reference
Primary Asset
```

用于资源加载、管理和打包。

------

## 15. 世界与关卡系统

UE 的世界结构大致是：

```text
World
└─ Level
   └─ Actor
      └─ Component
```

UE5 中还有：

```text
World Partition
Data Layer
Level Instance
Streaming Level
One File Per Actor
```

`World Partition` 常用于开放世界，把大地图自动分块加载。

------

## 16. 网络系统 Networking

UE 支持客户端/服务器架构。

核心概念：

```text
Server
Client
Replication
RPC
NetMulticast
Owner
Authority
PlayerController
PlayerState
GameState
```

常见写法：

```text
变量复制 Replicated
服务端 RPC Run on Server
客户端 RPC Run on Owning Client
多播 RPC Multicast
```

多人游戏里，很多逻辑必须区分：

```text
Has Authority
Is Locally Controlled
```

------

## 17. 音频系统 Audio

UE5 音频系统包括：

```text
Sound Wave
Sound Cue
MetaSounds
Audio Component
Attenuation
Sound Class
Sound Mix
Submix
Reverb
```

其中 `MetaSounds` 类似音频版的材质节点系统，可以用节点生成和控制声音。

------

## 18. Niagara 特效系统

Niagara 是 UE5 的粒子和特效系统。

用于：

```text
火焰
烟雾
爆炸
魔法
拖尾
命中特效
环境粒子
技能特效
```

主要资源：

```text
Niagara System
Niagara Emitter
Niagara Module
```

------

## 19. 反射系统 Reflection

这是 UE C++ 很核心的一部分。

常见宏：

```cpp
UCLASS()
USTRUCT()
UENUM()
UFUNCTION()
UPROPERTY()
GENERATED_BODY()
```

反射系统让 C++ 类型可以被：

```text
蓝图访问
编辑器显示
序列化保存
网络复制
垃圾回收
属性面板编辑
```

比如：

```cpp
UPROPERTY(EditAnywhere, BlueprintReadWrite)
float HP;
```

这个变量才能在编辑器和蓝图中显示和使用。

------

## 20. 插件系统 Plugins

UE5 很多功能都是插件形式存在的，比如：

```text
Enhanced Input
Gameplay Ability System
Common UI
Niagara
Control Rig
Modeling Tools
Water
Chaos Vehicles
MetaHuman
PCG
```

插件可以是：

```text
引擎插件
项目插件
运行时插件
编辑器插件
```

------

# 从游戏对象角度看 UE5 的核心组成

如果按实际开发中最常接触的对象层级，可以记成：

```text
GameInstance
↓
World
↓
Level
↓
GameMode / GameState
↓
PlayerController / AIController
↓
Pawn / Character
↓
ActorComponent / SceneComponent
↓
Mesh / Camera / Collision / Movement / Audio
```

一个第三人称项目中通常会有：

```text
BP_GameMode
BP_PlayerController
BP_PlayerCharacter
BP_EnemyCharacter
BP_AIController
BT_Enemy
BB_Enemy
ABP_Player
WBP_HUD
GA_Attack
GE_Damage
```

------

# 简单总结

UE5 由很多系统组成，但最核心可以记这几类：

```text
编辑器：创建内容
运行时：运行游戏
游戏框架：Actor、Pawn、Character、Controller
组件系统：Mesh、Collision、Camera、Movement
蓝图系统：可视化逻辑
C++ 模块系统：底层代码
渲染系统：画面
动画系统：角色动作
物理系统：碰撞和模拟
输入系统：玩家操作
AI 系统：敌人行为
UI 系统：界面
资源系统：管理资产
网络系统：多人同步
反射系统：让 C++ 和编辑器/蓝图互通
插件系统：扩展功能
```

对初学 UE5 来说，最先重点理解这几层就够了：

```text
Actor
Component
Pawn / Character
Controller
GameMode
Blueprint
Anim Blueprint
UMG
Input
Collision
```