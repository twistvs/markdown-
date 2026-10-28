在 UE5 中，行为树（Behavior Tree）是一套用于实现 AI 决策与行为控制的系统。

它的核心思想是：**通过树状结构组织 AI 的决策流程，由条件节点决定执行哪个分支，由任务节点执行具体行为。**

对于 UE5 游戏客户端面试，建议重点掌握以下 9 个核心类：

| 核心类                        | 主要职责                   |
| ----------------------------- | -------------------------- |
| `AAIController`               | 控制 AI，启动行为树        |
| `UBehaviorTree`               | 保存行为树的结构和节点配置 |
| `UBehaviorTreeComponent`      | 负责行为树的运行与调度     |
| `UBlackboardData`             | 定义黑板中有哪些 Key       |
| `UBlackboardComponent`        | 保存运行时黑板数据         |
| `UBTNode`                     | 行为树节点的基类           |
| `UBTCompositeNode`            | 组合节点，控制执行流程     |
| `UBTTaskNode`                 | 任务节点，执行具体行为     |
| `UBTDecorator` / `UBTService` | 条件判断与辅助更新         |

## 一、首先理解行为树的整体架构

假设我们要实现一个敌人 AI：

- 如果发现玩家，就追击并攻击。
- 如果没有发现玩家，就在出生点附近巡逻。
- 如果生命值过低，就优先逃跑。

可以设计成：

```
                  AAIController
                        │
                        │ RunBehaviorTree()
                        ▼
               UBehaviorTreeComponent
                        │
                        │ 执行
                        ▼
                  UBehaviorTree
                        │
                       Root
                        │
                     Selector
                        │
             ┌──────────┼──────────┐
             │          │          │
         Sequence    Sequence   Sequence
           逃跑        战斗        巡逻
             │          │          │
         Decorator   Decorator   Task
         血量过低     发现玩家     随机寻点
             │          │          │
           Task        Task       Task
           MoveTo      MoveTo     MoveTo
```

同时，行为树会通过 Blackboard 查询 AI 的状态：

```
             UBlackboardData
              （定义 Key）
                    │
                    ▼
           UBlackboardComponent
              （保存实际值）
                    │
         ┌──────────┼──────────┐
         │          │          │
    TargetActor   Health   PatrolLocation
      玩家A        30       (100,200,0)
```

这里最重要的是区分：

- `UBehaviorTree` 是**行为配置资源。**
- `UBehaviorTreeComponent` 是**行为执行器。**
- `UBlackboardData` 是**黑板数据定义。**
- `UBlackboardComponent` 是**黑板运行时实例。**

这些概念对应 UE5 行为树系统中资源和运行时组件的分离设计。

## 二、AAIController：AI 的控制者

`AAIController` 继承自 `AController`，负责控制 AI Pawn。

它本身并不是行为树节点，而是游戏 AI 系统的重要入口。

主要职责包括：

1. Possess 一个 AI Pawn。
2. 启动行为树。
3. 管理 AI 的移动请求。
4. 设置 AI 的注视目标。
5. 与 AI Perception、黑板等系统交互。

### 如何启动行为树？

创建自定义 AIController：

```c++
UCLASS()
class AEnemyAIController : public AAIController
{
    GENERATED_BODY()

public:
    UPROPERTY(EditDefaultsOnly, Category = "AI")
    TObjectPtr<UBehaviorTree> BehaviorTreeAsset;

protected:
    virtual void OnPossess(APawn* InPawn) override;
};

void AEnemyAIController::OnPossess(APawn* InPawn)
{
    Super::OnPossess(InPawn);

    if (BehaviorTreeAsset)
    {
        RunBehaviorTree(BehaviorTreeAsset);
    }
}
```

其中：

```c++
RunBehaviorTree(BehaviorTreeAsset);
```

会启动行为树执行，并使用相应的 BehaviorTreeComponent 和 BlackboardComponent。

通过：

```c++
UBlackboardComponent* BB = GetBlackboardComponent();
```

即可取得黑板组件。

例如：

```c++
BB->SetValueAsObject(
    TEXT("TargetActor"),
    PlayerActor
);
```

这样就可以更新 AI 的目标。

**注意：AIController 负责控制 AI，但实际行为树节点的调度不是由 AIController 自己完成的，而是由 UBehaviorTreeComponent 完成。** 

## 三、UBehaviorTree 和 UBehaviorTreeComponent

这两个类特别容易混淆。

### 1. UBehaviorTree：行为树资源

`UBehaviorTree` 主要保存：

- 行为树根节点。
- 节点之间的层级关系。
- 使用的 Blackboard Asset。
- 节点的配置和执行规则。

你在编辑器里创建的 `BT_Enemy`，本质上就是一个 UBehaviorTree 资源。

假设场景中有 100 个相同类型的敌人，那么它们可以使用同一个 `BT_Enemy`。

**不需要为每个敌人复制一份行为树资源。**

### 2. UBehaviorTreeComponent：行为树运行时组件

`UBehaviorTreeComponent` 继承自 `UBrainComponent`。

它负责实际执行行为树，包括：

- 启动、停止、重启行为树。
- 记录当前正在执行的节点。
- 管理任务执行结果。
- 处理 Decorator 引发的执行流中断。
- 调度 Service。
- 保存节点运行时状态和内存。

例如：

```c++
UBehaviorTreeComponent* BTC =
    Cast<UBehaviorTreeComponent>(GetBrainComponent());

if (BTC)
{
    BTC->StopTree(EBTStopMode::Safe);
}
```

核心接口还包括：

```c++
StartTree(...)
StopTree(...)
RestartTree(...)
```

这意味着：

**100 个 AI 可以共享一个 UBehaviorTree 资源，但各自有独立的 UBehaviorTreeComponent 执行状态。** 

## 四、UBlackboardData 和 UBlackboardComponent

这两个类的关系，实际上和前面那两个非常相似。

### 1. UBlackboardData：黑板定义

假设我们在编辑器里创建了一个 `BB_Enemy`：

| Key            | 类型   |
| -------------- | ------ |
| TargetActor    | Object |
| CanAttack      | Bool   |
| PatrolLocation | Vector |
| Health         | Float  |

`UBlackboardData` 定义这些 Key 的名称、类型和其他元数据。

但它不是用来保存每个敌人运行时血量的对象。

### 2. UBlackboardComponent：黑板实例

每个 AI 通常有自己的 BlackboardComponent。

例如：

```
                  BB_Enemy
              UBlackboardData
                     │
          ┌──────────┴──────────┐
          │                     │
     Enemy A                 Enemy B
 UBlackboardComponent    UBlackboardComponent
          │                     │
     Health = 80            Health = 20
     Target = Player        Target = nullptr
```

因此，两个敌人虽然使用相同的黑板定义，但运行时的数据可以完全不同。

需要注意，UE5 也支持 Blackboard Key 的 `Instance Synced` 设置，使部分 Key 能在符合条件的黑板实例之间同步，但普通 Key 默认不是所有 AI 共享的。

### 常见黑板操作

```c++
UBlackboardComponent* BB =
    OwnerComp.GetBlackboardComponent();

// 写入
BB->SetValueAsBool(TEXT("CanAttack"), true);

BB->SetValueAsVector(
    TEXT("PatrolLocation"),
    FVector(100, 200, 0)
);

// 读取
bool bCanAttack =
    BB->GetValueAsBool(TEXT("CanAttack"));

// 清空
BB->ClearValue(TEXT("TargetActor"));
```

黑板本质上是 AI 的工作记忆，而行为树根据这些数据决定执行什么动作。

## 五、UBTNode：所有行为树节点的基类

前面介绍的是行为树的管理类，接下来是行为树真正执行逻辑的节点类。

它们的继承关系如下：

```
UObject
   │
   └── UBTNode
         │
         ├── UBTCompositeNode
         │      ├── UBTComposite_Selector
         │      ├── UBTComposite_Sequence
         │      └── UBTComposite_SimpleParallel
         │
         ├── UBTTaskNode
         │      ├── UBTTask_MoveTo
         │      ├── UBTTask_Wait
         │      └── UBTTask_BlueprintBase
         │
         └── UBTAuxiliaryNode
                ├── UBTDecorator
                │      └── UBTDecorator_Blackboard
                │
                └── UBTService
                       └── UBTService_BlackboardBase
```

其中 `UBTNode` 主要提供通用的节点功能，包括初始化、节点内存管理等。

下面的四类节点是行为树中最重要的部分。

## 六、UBTCompositeNode：组合节点

组合节点负责**控制子节点的执行顺序，以及根据子节点的执行结果决定下一步做什么。**

UE5 有三种主要的组合节点。

### 1. Selector（选择器）

对应 C++ 类：

```
UBTComposite_Selector
```

特点：

**从左向右尝试子节点，直到某个子节点成功。如果所有子节点都失败，Selector 才失败。**

例如：

```
             Selector
                │
      ┌─────────┼─────────┐
      │         │         │
    逃跑       攻击       巡逻
```

假设：

- 逃跑条件不满足，返回 Failed。
- 攻击条件满足，执行攻击并返回 Succeeded。
- 巡逻分支不再执行。

可以把它理解为带有优先级的 `if-else`。

但如果攻击任务返回 `InProgress`，Selector 会等待该任务继续执行，而不是立刻尝试巡逻。

### 2. Sequence（顺序节点）

对应 C++ 类：

```
UBTComposite_Sequence
```

特点：

从左到右依次执行子节点，全部成功才成功，任意一个失败则整个 Sequence 失败。

例如：

```
             Sequence
                │
       ┌────────┼────────┐
       │        │        │
    寻找敌人   靠近敌人   发动攻击
```

假设：

- 寻找敌人成功。
- 靠近敌人成功。
- 发动攻击失败。

那么整个 Sequence 返回 Failed。

Selector 和 Sequence 的区别可以记成：

|              | Selector           | Sequence          |
| ------------ | ------------------ | ----------------- |
| 核心逻辑     | 找到一个成功的分支 | 要求所有步骤成功  |
| 遇到 Success | 停止，返回 Success | 继续执行下一个    |
| 遇到 Failed  | 尝试下一个         | 停止，返回 Failed |
| 全部完成     | 全部失败才失败     | 全部成功才成功    |

这两个节点是设计行为树时最常用的组合节点。

### 3. Simple Parallel（简单并行）

对应 C++ 类：

```
UBTComposite_SimpleParallel
```

允许一个主 Task 运行时，同时执行一个后台子树。

例如：

```
          Simple Parallel
                │
        ┌───────┴──────────┐
        │                  │
     Main Task       Background Branch
     播放攻击动作         调整朝向
```

它不同于多线程并行计算，而是行为树执行逻辑上的并行。

主任务结束后，可以选择立即终止后台分支，或者允许后台分支执行完成。

## 七、UBTTaskNode：任务节点

`UBTTaskNode` 是行为树真正执行动作的节点。

例如：

- Move To：移动到目标位置。
- Wait：等待指定时间。
- Play Animation：播放动画。
- 自定义 Attack：发动攻击。
- 自定义 FindPatrolPoint：寻找巡逻位置。

### 1. Task 最重要的函数

```c++
virtual EBTNodeResult::Type ExecuteTask(
    UBehaviorTreeComponent& OwnerComp,
    uint8* NodeMemory
) override;
```

这个函数用于启动任务。

返回值有三种需要重点掌握：

| 返回值       | 含义                       |
| ------------ | -------------------------- |
| `Succeeded`  | 任务成功完成               |
| `Failed`     | 任务执行失败               |
| `InProgress` | 任务还在执行，暂时不能结束 |

例如：

```
EBTNodeResult::Type UMyTask::ExecuteTask(
    UBehaviorTreeComponent& OwnerComp,
    uint8* NodeMemory)
{
    return EBTNodeResult::Succeeded;
}
```

这表示任务立即执行成功。

### 2. 如果任务需要持续执行怎么办？

例如一个攻击任务：

```
开始攻击
   ↓
播放攻击蒙太奇
   ↓
等待动画完成
   ↓
结束任务
```

攻击并不是瞬间完成的，所以执行时应该返回：

```
return EBTNodeResult::InProgress;
```

当蒙太奇播放完成后，通过回调调用：

```
FinishLatentTask(
    OwnerComp,
    EBTNodeResult::Succeeded
);
```

这样行为树才知道该 Task 已经结束，可以继续执行后续节点。

这也是异步任务的重要机制。

![img](../../图片/dev.epicgames.png)

Epic Developer Community



### 3. Task 的生命周期

主要关注三个函数：

```
ExecuteTask()  // 开始执行任务

TickTask()     // 任务运行期间的 Tick（需要启用）

AbortTask()    // 任务被要求中断
```

例如敌人正在追击玩家：

```
MoveTo 玩家
    │
    │ InProgress
    ▼
玩家进入攻击范围
    │
    ▼
追击任务被中断
    │
    ▼
AbortTask()
    │
    ▼
执行攻击任务
```

这里需要注意：

如果 `AbortTask()` 能同步完成中断，可以直接返回 `Aborted`；如果需要异步清理，可以返回 `InProgress`，并在清理完成后调用 `FinishLatentAbort()`。

如果使用 TickTask，需要正确开启 Tick 通知。对于等待动画、移动等异步操作，通常更推荐使用完成事件或 AI 消息结束任务，而不是每帧轮询。

## 八、UBTDecorator：装饰器节点

`UBTDecorator` 主要负责判断分支是否允许执行，以及在条件变化时触发中断。

例如：

```
          Sequence：攻击玩家
                 │
          Decorator
        CanAttack == true
                 │
            Task：Attack
```

当 `CanAttack == false` 时，攻击分支不能进入。

### 1. 自定义 Decorator

常用函数：

```
virtual bool CalculateRawConditionValue(
    UBehaviorTreeComponent& OwnerComp,
    uint8* NodeMemory
) const override;
```

返回 true 表示原始条件满足，false 表示不满足；最终判断还可能受到 Decorator 的反转设置影响。

### 2. Observer Aborts 是什么？

这部分是 UE5 行为树面试的重点。

假设行为树如下：

```
                Selector
                   │
          ┌────────┴────────┐
          │                 │
       追击玩家            巡逻
          │
      Decorator
    HasTarget == true
```

一开始 `HasTarget = false`，AI 正在巡逻。

突然玩家进入视野，黑板的 `HasTarget` 变成了 true。

问题来了：

AI 会不会立即停止巡逻，开始追击玩家？

不一定。需要看 Decorator 是否在观察条件变化，以及 `Observer Aborts` 如何设置。

| Observer Aborts | 作用                                   |
| --------------- | -------------------------------------- |
| None            | 不主动中断当前执行分支                 |
| Self            | 当自身分支条件不再满足时，中断自身分支 |
| Lower Priority  | 当条件满足时，可以中断右侧低优先级分支 |
| Both            | 同时支持 Self 和 Lower Priority        |

因此，如果希望 AI 在发现玩家后立即从巡逻切换到追击，可以将追击分支的 Blackboard Decorator 设置为 `Both`，并正确配置条件观察。

对于 Blackboard Decorator，黑板 Key 的变化可以触发条件重新评估，而无需每帧遍历整棵行为树。

![img](../../图片/dev.epicgames.png)

Epic Developer Community



## 九、UBTService：服务节点

`UBTService` 主要负责在分支处于激活状态时，周期性执行辅助逻辑。

例如：

- 检查玩家距离。
- 更新攻击范围状态。
- 寻找更合适的目标。
- 更新黑板中的某些数据。

假设：

```
                 Selector
                    │
           Service：UpdateTarget
                    │
          ┌─────────┴─────────┐
          │                   │
       追击玩家              巡逻
```

Service 可以按照设定的间隔执行，例如每 0.2 秒更新一次目标信息。

### 常见函数

```
// 分支开始相关执行时
OnBecomeRelevant()

// 按设定间隔执行
TickNode()

// 分支不再相关时
OnCeaseRelevant()
```

其中：

```
virtual void TickNode(
    UBehaviorTreeComponent& OwnerComp,
    uint8* NodeMemory,
    float DeltaSeconds
) override;
```

可以用于更新黑板。

Service 和 Decorator 最大的区别是：

- Service 负责更新信息。
- Decorator 负责根据条件控制执行流程。

Service 自身不通过返回 Success、Failed 来选择分支，它通常是通过更新 Blackboard，让 Decorator 决定是否中断当前分支。

![img](../../图片/dev.epicgames.png)

Epic Developer Community



## 十、实际开发中，整个行为树是怎么运行的？

以追击玩家为例。

假设敌人使用了 AI Perception 感知系统：

```
玩家进入敌人视野
        │
        ▼
UAIPerceptionComponent
        │
        │ 感知事件
        ▼
AAIController
        │
        │ 更新 TargetActor
        ▼
UBlackboardComponent
        │
        │ 黑板 Key 发生变化
        ▼
UBTDecorator_Blackboard
        │
        │ 追击条件成立
        ▼
UBehaviorTreeComponent
        │
        │ 中断巡逻，执行追击分支
        ▼
UBTTask_MoveTo
        │
        ▼
敌人向玩家移动
```

这里的职责非常清楚：

- AI Perception 负责感知。
- AIController 负责协调控制。
- Blackboard 负责保存目标信息。
- Decorator 负责判断和中断。
- BehaviorTreeComponent 负责调度。
- Task 负责执行移动。

这也体现了 UE5 行为树的一个重要特点：它并不是每帧从 Root 开始重新遍历整棵树，而是大量使用事件驱动的执行流更新。 当然，运行中的 Task、Service 仍然可以根据需要进行 Tick。

![img](../../图片/dev.epicgames.png)

Epic Developer Community



## 十一、一个很容易被忽略的知识点：节点实例和 NodeMemory

这是 C++ 面试中比较有深度的追问。

假设有 100 个敌人，它们全部使用同一个行为树资源。

那么每个敌人都会有一套独立的 Task UObject 吗？

默认情况下，并不会。

UE5 的原生行为树节点默认采用共享模板节点的方式，多个 AI 可以使用相同的节点 UObject，以降低内存开销。

因此，不能随意在未实例化的 Task 成员变量里保存某个 AI 的运行时状态。

例如下面的代码可能出现问题：

```
UCLASS()
class UBTTask_Attack : public UBTTaskNode
{
    GENERATED_BODY()

    // 危险：默认共享节点上保存特定 AI 的状态
    AActor* CurrentTarget;
};
```

如果多个敌人共享这个节点，`CurrentTarget` 就不是天然属于某个敌人的独立变量。

UE5 提供了两种常见解决方式：

方式一：使用 NodeMemory

通过 `GetInstanceMemorySize()` 为每个行为树运行实例分配节点私有数据。

例如：

```
struct FAttackTaskMemory
{
    float AttackTime = 0.f;
    bool bStarted = false;
};
```

每个 AI 的 BehaviorTreeComponent 都可以拥有相应的独立 NodeMemory。

方式二：启用节点实例化

```
bCreateNodeInstance = true;
```

这样每个运行该行为树的 AI 都可以获得自己的节点 UObject 实例。

代价是需要更多内存和对象管理开销。

这一机制在官方 `UBTTaskNode` 文档中也有明确说明。

![img](../../图片/dev.epicgames.png)

Epic Developer Community



## 十二、面试应该怎么回答？

> UE5 行为树的核心类可以分为管理类和节点类。
>
> 管理类主要有 AAIController、UBehaviorTree、UBehaviorTreeComponent，以及 Blackboard 相关的类。
>
> AAIController 负责控制 AI Pawn，并通过 RunBehaviorTree 启动行为树。
>
> UBehaviorTree 是行为树资源，保存节点结构和配置；UBehaviorTreeComponent 是实际运行行为树的组件，负责节点调度、任务执行和分支中断。
>
> Blackboard 则分为 UBlackboardData 和 UBlackboardComponent，前者定义数据结构，后者保存每个 AI 的运行时黑板数据。
>
> 在节点层面，所有节点都继承自 UBTNode，主要分为 Composite、Task、Decorator 和 Service。
>
> Composite 负责执行流程，例如 Selector 和 Sequence；Task 负责执行具体动作，例如移动、攻击；Decorator 负责条件判断和分支中断；Service 负责周期性更新辅助数据。
>
> 另外，UE5 行为树采用事件驱动优化，Decorator 可以观察黑板变化并中断分支，而不是要求每帧重新遍历整棵树。
>
> 对于 C++ 自定义节点，还需要注意原生行为树节点默认可能由多个 AI 共享，不能随意在节点 UObject 中保存 AI 私有运行时状态，可以使用 NodeMemory 或启用节点实例化。

### 建议重点掌握的三个面试追问

1. Selector 和 Sequence 有什么区别？ 尤其是返回 `InProgress` 时，执行流程如何变化。
2. Task 返回 InProgress 后，怎么通知行为树任务完成？ 重点掌握 `FinishLatentTask()` 和 `AbortTask()`。
3. 为什么 UE5 默认共享行为树节点？NodeMemory 是为了解决什么问题？ 重点理解内存优化与 AI 独立运行状态之间的关系。

对于游戏客户端面试，前两个属于基础知识，第三个更能体现对 UE5 行为树底层实现的理解。