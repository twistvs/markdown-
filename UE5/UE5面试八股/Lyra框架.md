我看一下当前工程的模块和插件结构，再按功能拆开介绍，方便你建立整体认识。

Lyra 可以理解为一个 **基于 UE5 的模块化多人游戏示例工程**。它既包含射击、技能、角色动画这些玩法，也包含菜单、联网、设置、资源加载等完整游戏需要的基础设施。

结合你当前的 UE5.3 工程，学习时可以分成 **8 块**。这是按功能划分，和源码目录不是严格一一对应。

### 1. 游戏框架与玩法装配

这部分决定“当前运行什么玩法、加载哪些功能、玩家使用什么角色”。

主要包括：

- **GameMode / GameState**：游戏规则与全局状态。
- **Experience**：配置一次游戏体验所需的 PawnData、Game Feature 插件和扩展动作等。
- **PawnData**：配置角色类、输入配置、技能集等。
- **Game Features**：以插件形式装配玩法，可添加组件、授予技能、扩展 UI 等。
- **初始化状态机制**：协调角色、输入、技能系统的初始化顺序。

主要目录：`GameModes`、`GameFeatures`、`System`。

**Experience 是理解 Lyra 架构的重要入口**：同一套基础框架，通过不同配置和插件组合出不同玩法。

### 2. 玩家、角色与输入

这部分负责“玩家控制谁，以及如何控制”。

| 内容                         | 主要职责                            |
| ---------------------------- | ----------------------------------- |
| `LyraPlayerController`       | 玩家控制、输入与客户端相关逻辑      |
| `LyraPlayerState`            | 玩家状态，持有 Lyra 的 ASC          |
| `LyraCharacter`              | 游戏中的角色实体                    |
| `LyraPawnExtensionComponent` | 协调 PawnData、ASC 绑定与角色初始化 |
| `LyraHeroComponent`          | 英雄角色的输入和相机等接入          |
| Enhanced Input               | 输入动作、映射和输入上下文          |

主要目录：`Player`、`Character`、`Input`。

你引用的 **`Character.cpp` 是 UE 引擎中 `ACharacter` 的实现**。它与 Lyra 的关系是：

```text
引擎 ACharacter
    └─ ALyraCharacter
          └─ 具体角色蓝图
```

移动、跳跃等底层能力由引擎提供，Lyra 在其上接入技能、输入、血量、相机等系统。

### 3. 技能、属性与伤害：GAS

这部分负责“角色能做什么，以及做完产生什么效果”。

- **ASC**：管理技能、效果、属性和 Gameplay Tag。
- **GA**：执行技能，例如跳跃、开火、换弹、冲刺、死亡。
- **GE**：施加伤害、治疗、属性修改等效果。
- **AttributeSet**：保存生命值等属性。
- **AbilitySet**：批量配置并授予技能、效果和属性集。
- **Gameplay Tag**：标识状态、输入和事件，并参与技能激活、阻止、取消等判断。

主要目录：`AbilitySystem`。

Lyra 中 **ASC 通常属于 PlayerState，当前 Pawn 作为 AvatarActor**。理解这种关系，有助于理解死亡、重生时技能系统如何重新绑定角色。

### 4. 背包、装备与武器

这部分负责“玩家拥有什么、正在使用什么，以及武器如何工作”。

| 系统      | 负责什么                                 |
| --------- | ---------------------------------------- |
| Inventory | 物品定义、物品实例、数量等               |
| Equipment | 装备实例、装备时生成的 Actor、授予的技能 |
| QuickBar  | 快捷栏与当前装备切换                     |
| Weapons   | 武器参数、散布、射击检测、命中反馈等     |

主要目录：`Inventory`、`Equipment`、`Weapons`。

以步枪为例，可以理解为：

```text
背包中的步枪物品
    → 通过快捷栏选中
    → 装备系统创建武器装备实例
    → 授予开火、换弹等 GA
    → 开火 GA 执行射击逻辑
```

**物品实例、装备实例、场景里的枪模型 Actor，是不同对象。**

### 5. 角色表现：动画、相机与反馈

这部分让玩法具备可见、可听的表现。

- **Animation**：移动动画、动画层、持枪姿态、瞄准与转向等。
- **Camera**：相机模式、模式切换与混合、相机穿透处理等。
- **Cosmetics**：角色外观和展示部件。
- **Audio**：声音播放与混音等支持。
- **Feedback**：震动等反馈。
- **Niagara、材质与 Gameplay Cue**：表现枪口火焰、命中、技能效果等。

主要目录：`Animation`、`Camera`、`Cosmetics`、`Audio`、`Feedback`，具体资源还分布在 `Content` 和玩法插件中。

### 6. UI、交互与消息

这部分负责“玩家看到什么，以及系统如何通知界面”。

- **CommonUI / CommonGame**：菜单、界面层级、输入焦点等。
- **UIExtension**：让玩法插件向指定位置插入 UI。
- **HUD**：血量、弹药、准星、比分等。
- **Interaction**：扫描交互对象、显示提示、执行交互。
- **GameplayMessageRouter**：通过消息广播连接系统，例如击杀消息驱动击杀提示。

主要目录：`UI`、`Interaction`、`Messages`。

例如击杀发生后，战斗系统广播消息，UI 监听消息更新提示，可以减少战斗代码与具体 Widget 的直接依赖。

### 7. 多人游戏与产品基础设施

这部分负责“让游戏能作为一个完整产品运行”。

包括：

- 网络复制、RPC、技能预测。
- 用户登录、在线会话与加入游戏。
- 队伍分配与队伍信息。
- 游戏设置、画质、音量与输入设置。
- 加载界面与资源管理。
- 性能统计、回放、热更新支持。
- 编辑器工具与自动化测试。

主要分布在 `Teams`、`Settings`、`Performance`、`Replays`、`Hotfix`、`Development`，以及 `CommonUser`、`CommonLoadingScreen` 等插件。

**联网是贯穿各系统的能力**：角色、技能、装备和比赛状态都有各自的网络处理。

### 8. 具体玩法与内容插件

前面的系统提供基础能力，这些插件把它们组合成可以玩的内容。

| 插件              | 内容定位                                                 |
| ----------------- | -------------------------------------------------------- |
| `ShooterCore`     | 射击玩法核心，包含武器、战斗技能、团队死斗和控制点等内容 |
| `ShooterMaps`     | 射击玩法地图资源                                         |
| `TopDownArena`    | 俯视角竞技场、放置炸弹等玩法                             |
| `ShooterExplorer` | 探索、交互、背包和地图等示例                             |
| `ShooterTests`    | 射击玩法相关测试                                         |

因此，看 Lyra 时需要同时看两处：**`Source/LyraGame` 中的通用 C++ 系统，以及 `Plugins/GameFeatures` 中的具体玩法代码、蓝图和配置。**

如果你准备从角色开始读代码，建议沿着这条主线：

**Experience / PawnData → 角色初始化 → 输入 → ASC 激活 GA → 武器或移动逻辑 → 动画与 UI 反馈。**

你已经在看 `Character.cpp` 和 GA，可以先跟通 **“按下跳跃 → `GA_Hero_Jump` → `ULyraGameplayAbility_Jump` → `ACharacter::Jump()` → CharacterMovement 执行跳跃”**，再用同样的方法跟开火流程。