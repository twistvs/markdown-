在 UE5 中，**如果希望一个 `UObject` 跨场景（Level）切换后仍然存在**，最常见的方案是：

**将 UObject 创建在 GameInstance 下，并让 GameInstance 持有它的强引用。**

因为 `UGameInstance` 的生命周期贯穿整个游戏运行过程，不会因为切换关卡而销毁。

不过，这里涉及两个需要区分的概念：

1. **Outer（外部对象）**：决定 UObject 的对象归属关系，并影响对象生命周期管理。
2. **GC 引用关系**：决定 UObject 是否会被垃圾回收。

仅仅指定 `Outer` 为 `GameInstance`，并不能保证 UObject 一定不会被 GC 回收。

## 一、方案一：通过 GameInstance 保存 UObject

假设有一个 `UPlayerData`，用于保存玩家的金币、等级等数据，希望玩家切换地图后这些数据仍然存在。

### 1. 创建 UObject 类

```c++
UCLASS()
class UPlayerData : public UObject
{
    GENERATED_BODY()

public:
    UPROPERTY()
    int32 Gold = 100;

    UPROPERTY()
    int32 Level = 1;
};
```

### 2. 自定义 GameInstance

```c++
UCLASS()
class UMyGameInstance : public UGameInstance
{
    GENERATED_BODY()

public:
    UPROPERTY()
    TObjectPtr<UPlayerData> PlayerData;

    virtual void Init() override;
};
```

在 `Init()` 中创建对象：

```c++
void UMyGameInstance::Init()
{
    Super::Init();

    PlayerData = NewObject<UPlayerData>(this);
}
```

这里有两个关键点：

- `NewObject<UPlayerData>(this)`：将 `GameInstance` 设置为对象的 Outer。
- `UPROPERTY()` + `TObjectPtr`：使 GameInstance 对 PlayerData 的引用可以被 UE 的 GC 系统追踪，从而让它保持存活。

### 3. 切换场景

```c++
UGameplayStatics::OpenLevel(
    GetWorld(),
    FName("NewLevel")
);
```

关卡切换之后，可以重新获取 `PlayerData`：

```C++
UMyGameInstance* GI =
    Cast<UMyGameInstance>(GetGameInstance());

if (GI && GI->PlayerData)
{
    UE_LOG(LogTemp, Warning, TEXT("Gold: %d"),
        GI->PlayerData->Gold);
}
```

只要 `GameInstance` 仍然存在，并且保持对 `PlayerData` 的强引用，`PlayerData` 就不会因普通的关卡切换而销毁。

还需要在 Project Settings → Maps & Modes → Game Instance Class 中，把自定义的 `UMyGameInstance` 设置为实际使用的类。

## 二、方案二：使用 GameInstanceSubsystem（更适合管理系统）

如果这个 UObject 不只是保存数据，还负责一些长期存在的业务逻辑，例如背包管理、任务管理、玩家数据管理，那么更推荐使用 `UGameInstanceSubsystem`。

```C++
UCLASS()
class UInventorySubsystem : public UGameInstanceSubsystem
{
    GENERATED_BODY()

public:
    UPROPERTY()
    int32 Gold = 100;

    virtual void Initialize(
        FSubsystemCollectionBase& Collection) override
    {
        Super::Initialize(Collection);
    }

    virtual void Deinitialize() override
    {
        Super::Deinitialize();
    }
};
```

使用时：

```C++
UInventorySubsystem* Inventory =
    GetGameInstance()->GetSubsystem<UInventorySubsystem>();
```

`UGameInstanceSubsystem` 本身就是 UObject，并且具备以下特点：

- 自动创建，不需要手动 `NewObject`。
- 生命周期与 GameInstance 绑定。
- 切换 Level 不会销毁。
- 可以通过 `GetSubsystem` 获取。
- 支持 `Initialize` 和 `Deinitialize` 生命周期回调。

这是 UE5 官方提供的长生命周期管理机制。

## 三、其他方案

| 方案                   | 能否跨 Level            | 适用情况                 |
| ---------------------- | ----------------------- | ------------------------ |
| GameInstance + UObject | 可以                    | 保存玩家数据、运行时对象 |
| GameInstanceSubsystem  | 可以                    | 背包、任务等全局系统     |
| LocalPlayerSubsystem   | 通常可以                | 本地玩家专属系统         |
| WorldSubsystem         | 不可以跨普通 World 切换 | 当前世界的管理逻辑       |
| AddToRoot()            | 可以防止 GC 回收        | 特殊生命周期管理         |
| SaveGame               | 可以持久化数据          | 退出游戏后还需恢复的数据 |

其中 `AddToRoot()` 的作用是将 UObject 加入 GC 根集合：

```C++
MyObject->AddToRoot();

// 不再需要时
MyObject->RemoveFromRoot();
```

但是它只解决垃圾回收问题，不负责处理跨场景资源引用。如果忘记 `RemoveFromRoot()`，还可能造成对象长期无法释放，因此通常不建议优先使用这种方案。

## 四、需要注意的问题

**UObject 自身能够跨场景存在，不代表它引用的所有对象也能够跨场景存在。**

假设 `PlayerData` 里保存了：

```C++
UPROPERTY()
TObjectPtr<ACharacter> PlayerCharacter;
```

即使 `PlayerData` 没有被销毁，原关卡的 `PlayerCharacter` 仍然可能在切换 Level 时被销毁。

因此，跨场景 UObject 最好不要长期保存旧关卡的 Actor、Component、World 等引用。关卡切换后应重新获取或绑定这些对象。

另外，在多人游戏中，`GameInstance` 不会自动进行网络复制。服务端和客户端分别拥有自己的 GameInstance，不能依靠它直接同步数据。

## 五、面试时可以这样回答

> 如果希望一个 UObject 跨场景存在，我会考虑将它放在 GameInstance 中管理。
>
> 因为 GameInstance 的生命周期贯穿整个游戏运行过程，不会随着 OpenLevel 切换关卡而销毁。
>
> 具体实现上，我可以通过 NewObject 创建 UObject，并将 Outer 设置为 GameInstance，同时在 GameInstance 中使用 UPROPERTY 保存强引用，避免对象被 GC 回收。
>
> 如果这个 UObject 本身是一个长期存在的管理系统，我会优先考虑 GameInstanceSubsystem，因为它可以自动管理对象的创建和生命周期。
>
> 需要注意的是，跨场景 UObject 不应该继续使用旧场景中已经销毁的 Actor 或 Component。

**面试中还有一个很容易被追问的问题：**

`NewObject<UPlayerData>(GameInstance)` 已经将 Outer 设置为 GameInstance 了，为什么还需要使用 `UPROPERTY` 保存它？

这里考查的正是 **UE5 中 Outer 与垃圾回收引用关系的区别**。Outer 表示对象的归属关系，但并不等同于 GC 强引用；真正决定普通 UObject 是否能够存活的，是它是否能从 GC 根集合通过有效强引用到达。