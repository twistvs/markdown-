下面回答 **三、UE C++ 高频问题**。这一部分非常重要，UE5 实习岗经常会考，因为 UE C++ 和普通 C++ 不完全一样，尤其是 **反射系统、宏、UPROPERTY、UObject 生命周期、GC、NewObject、SpawnActor** 这些内容。

------

# 三、UE C++ 高频问题

## 1. UCLASS、USTRUCT、UFUNCTION、UPROPERTY 是什么？

这几个宏都和 UE 的**反射系统**有关。

UE 的 C++ 不是纯标准 C++，它在 C++ 基础上增加了一套反射系统，使得 C++ 类型、变量、函数可以被编辑器、蓝图、序列化、垃圾回收、网络复制等系统识别。

------

## 1.1 什么是反射系统？

普通 C++ 在运行时很难直接知道：

```cpp
class Player
{
public:
    int Health;
    void Attack();
};
```

比如程序运行时无法方便地问：

- 这个类有哪些变量？
- 这个类有哪些函数？
- 这个变量能不能在编辑器显示？
- 这个函数能不能被蓝图调用？
- 这个对象能不能被序列化？
- 这个 UObject 引用能不能被 GC 追踪？

UE 为了解决这些问题，引入了反射系统。

简单说：

> UE 反射系统让 C++ 类、结构体、函数、变量可以被引擎识别和管理。

------

## 1.2 UCLASS 是什么？

`UCLASS()` 用来标记一个类参与 UE 反射系统。

常见写法：

```cpp
UCLASS()
class MYPROJECT_API AMyCharacter : public ACharacter
{
    GENERATED_BODY()
};
```

它通常用于继承自 `UObject` 的类，例如：

- `AActor`
- `APawn`
- `ACharacter`
- `UActorComponent`
- `UUserWidget`
- `UDataAsset`
- `UGameInstance`
- `UAnimInstance`

如果一个类想被 UE 识别，通常需要加 `UCLASS()`。

------

## 1.3 GENERATED_BODY 是什么？

`GENERATED_BODY()` 是 UE 代码生成工具生成代码的插入位置。

UE 会通过 Unreal Header Tool，也就是 UHT，扫描这些宏，然后生成反射相关代码。

一般格式：

```cpp
UCLASS()
class MYPROJECT_API AMyActor : public AActor
{
    GENERATED_BODY()

public:
    AMyActor();
};
```

面试可以这样说：

> GENERATED_BODY 是 UE 自动生成反射代码的位置，配合 UCLASS、USTRUCT 等宏使用。如果没有它，类不能正常参与 UE 的反射和对象系统。

------

## 1.4 USTRUCT 是什么？

`USTRUCT()` 用来让结构体参与 UE 反射。

例如：

```cpp
USTRUCT(BlueprintType)
struct FWeaponData
{
    GENERATED_BODY()

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float Damage = 20.0f;

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float AttackRange = 150.0f;
};
```

这样这个结构体就可以：

- 在蓝图中使用
- 在编辑器中编辑
- 被序列化
- 作为 DataTable 行数据
- 作为函数参数暴露给蓝图

常见用途：

- 武器配置
- 技能数据
- 怪物属性
- 掉落信息
- 任务数据
- 存档数据

------

## 1.5 UFUNCTION 是什么？

`UFUNCTION()` 用来让函数参与 UE 反射。

例如：

```cpp
UFUNCTION(BlueprintCallable)
void Attack();
```

这样蓝图中就可以调用 `Attack()`。

常见参数：

```cpp
UFUNCTION(BlueprintCallable)
void Attack();
```

表示蓝图可以调用。

```cpp
UFUNCTION(BlueprintPure)
float GetHealthPercent() const;
```

表示蓝图纯函数，没有执行引脚，通常用于计算或获取值。

```cpp
UFUNCTION(BlueprintImplementableEvent)
void OnDead();
```

表示 C++ 声明函数，但具体实现交给蓝图。

```cpp
UFUNCTION(BlueprintNativeEvent)
void Interact();
```

表示 C++ 可以有默认实现，蓝图也可以重写。

------

## 1.6 UPROPERTY 是什么？

`UPROPERTY()` 用来让变量参与 UE 反射。

例如：

```cpp
UPROPERTY(EditAnywhere, BlueprintReadWrite)
float Health = 100.0f;
```

这个变量就可以：

- 在编辑器中显示
- 在蓝图中访问
- 被序列化
- 被 GC 追踪，如果它是 UObject 引用
- 被网络复制，如果加 Replicated

常见参数：

```cpp
UPROPERTY(EditAnywhere)
float Damage;
```

可以在编辑器中编辑。

```cpp
UPROPERTY(BlueprintReadOnly)
float Health;
```

蓝图可以读取，不能直接写。

```cpp
UPROPERTY(BlueprintReadWrite)
float Health;
```

蓝图可以读写。

```cpp
UPROPERTY(VisibleAnywhere)
UStaticMeshComponent* Mesh;
```

编辑器可见，但不允许随意替换。

------

## 1.7 面试推荐回答

> UCLASS、USTRUCT、UFUNCTION、UPROPERTY 都是 UE 反射系统相关的宏。UCLASS 用来标记 UObject 类，USTRUCT 用来标记结构体，UFUNCTION 用来让函数被蓝图、网络、反射系统识别，UPROPERTY 用来让变量被编辑器、蓝图、序列化、GC 和网络复制系统识别。它们的作用是让普通 C++ 代码接入 UE 引擎系统。

------

# 2. UPROPERTY 为什么重要？

`UPROPERTY` 是 UE C++ 里非常重要的宏，尤其和 **编辑器暴露、蓝图访问、序列化、垃圾回收** 有关。

------

## 2.1 UPROPERTY 的核心作用

`UPROPERTY` 主要有几个作用：

1. 暴露到编辑器
2. 暴露到蓝图
3. 支持序列化
4. 支持网络复制
5. 让 UObject 引用被 GC 追踪

其中面试最常问的是第 5 点：

> UObject 指针如果没有被 UPROPERTY 标记，可能不会被 UE 的 GC 正确追踪。

------

## 2.2 有 UPROPERTY 和没有 UPROPERTY 的区别

例如：

```cpp
UPROPERTY()
UObject* MyObject;
```

这个 `MyObject` 会被 UE 反射系统识别。

如果它指向一个 UObject，这个引用关系可以被 GC 追踪。

但是：

```cpp
UObject* MyObject;
```

这是普通 C++ 指针，UE 的 GC 不一定能通过它知道你还引用着这个对象。

结果可能是：

- 对象已经被 GC 回收
- 你的指针还保存着旧地址
- 再访问时出现野指针、崩溃

------

## 2.3 举例说明

错误示例：

```cpp
UCLASS()
class UMyManager : public UObject
{
    GENERATED_BODY()

public:
    UObject* CachedObject;
};
```

如果 `CachedObject` 指向某个 UObject，但没有 `UPROPERTY()`，GC 可能认为这个对象没人引用，从而回收它。

更安全写法：

```cpp
UCLASS()
class UMyManager : public UObject
{
    GENERATED_BODY()

public:
    UPROPERTY()
    UObject* CachedObject;
};
```

------

## 2.4 组件变量为什么通常要 UPROPERTY？

比如：

```cpp
UPROPERTY(VisibleAnywhere)
UStaticMeshComponent* MeshComp;
```

原因是：

- 需要在编辑器里显示
- 需要参与序列化
- 需要避免被 GC 误处理
- 需要让蓝图子类能看到组件

常见写法：

```cpp
UCLASS()
class AMyActor : public AActor
{
    GENERATED_BODY()

public:
    AMyActor();

protected:
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly)
    UStaticMeshComponent* MeshComp;
};
```

构造函数里：

```cpp
AMyActor::AMyActor()
{
    MeshComp = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("MeshComp"));
    RootComponent = MeshComp;
}
```

------

## 2.5 UPROPERTY 不等于智能指针

注意：

`UPROPERTY()` 不是普通 C++ 的智能指针，它不会自动 `delete` 对象。

它的作用是让 UE 的反射和 GC 系统知道：

> 这里有一个 UObject 引用关系。

UE 管理的是 UObject 对象，而不是普通 C++ 对象。

例如：

```cpp
UPROPERTY()
UObject* Obj;
```

这不是说 `Obj` 是智能指针，而是说 UE GC 可以追踪它。

------

## 2.6 面试推荐回答

> UPROPERTY 很重要，因为它可以让变量被 UE 的反射系统识别。它可以控制变量是否暴露到编辑器、蓝图，是否序列化，是否复制。对于 UObject 指针来说，UPROPERTY 还能让 GC 追踪引用关系，避免对象被错误回收。如果 UObject 指针没有 UPROPERTY 标记，就可能变成悬空指针或野指针。

------

# 3. UE 中的垃圾回收机制是什么？

UE 的垃圾回收主要针对 `UObject` 体系对象。

普通 C++ 中，我们常见：

```cpp
MyClass* Obj = new MyClass();
delete Obj;
```

但 UE 中很多对象是 `UObject`，不推荐手动 `delete`，而是由 UE 的 GC 管理。

------

## 3.1 UE GC 管理哪些对象？

UE GC 主要管理继承自 `UObject` 的对象，例如：

- `UObject`
- `UActorComponent`
- `UUserWidget`
- `UDataAsset`
- `UAnimInstance`
- 很多资源对象

`AActor` 也是 UObject 的子类，但 Actor 通常有自己的世界生命周期，销毁时一般使用：

```cpp
Destroy();
```

而不是手动 delete。

------

## 3.2 UE GC 如何判断对象是否可回收？

UE 的 GC 会从一批根对象开始，沿着引用关系查找仍然可达的对象。

常见引用来源：

- Root Set
- UPROPERTY 引用
- UObject 内部引用
- AddToRoot 的对象
- 正在被世界或引擎管理的对象

如果一个 UObject 不再被有效引用，GC 就可能在之后的回收阶段销毁它。

可以简单理解为：

```text
根对象
  ↓
UPROPERTY 引用
  ↓
其他 UObject
  ↓
仍然可达：保留

不可达对象：等待 GC 回收
```

------

## 3.3 Actor 怎么销毁？

Actor 通常不使用 `delete`。

正确方式：

```cpp
MyActor->Destroy();
```

`Destroy()` 的含义不是立刻像 C++ `delete` 一样释放内存，而是标记该 Actor 要被销毁，然后在合适时机由引擎处理。

常见流程：

```text
Destroy()
  ↓
标记 Pending Kill / Being Destroyed
  ↓
EndPlay
  ↓
从 World 中移除
  ↓
之后由 GC 清理内存
```

------

## 3.4 UObject 怎么销毁？

UObject 通常也不手动 delete。

如果你不再持有它的引用，GC 会在合适时机回收。

例如：

```cpp
UObject* Obj = NewObject<UObject>();
```

如果没有任何 UPROPERTY 或其他有效引用保存它，它之后可能被 GC 回收。

如果你想长期保存：

```cpp
UPROPERTY()
UObject* Obj;
```

或者让某个 UObject 成为它的 Outer，并确保 Outer 本身生命周期足够长。

------

## 3.5 AddToRoot 是什么？

`AddToRoot()` 可以把 UObject 加入 Root Set，防止被 GC。

例如：

```cpp
Obj->AddToRoot();
```

但是不推荐随便使用，因为如果忘记：

```cpp
Obj->RemoveFromRoot();
```

可能导致内存无法回收。

面试中可以说：

> AddToRoot 可以强制对象不被 GC，但一般不建议滥用，正常情况下应通过 UPROPERTY 或合理 Outer 管理引用。

------

## 3.6 UE GC 和 C++ 智能指针的区别

普通 C++ 对象常用：

```cpp
TSharedPtr
TWeakPtr
TUniquePtr
```

但 UObject 通常不使用 `TSharedPtr` 管理生命周期。

UE 中 UObject 常用：

```cpp
UPROPERTY()
UObject* Obj;
```

或者：

```cpp
TObjectPtr<UObject> Obj;
```

在 UE5 中，很多代码会看到：

```cpp
UPROPERTY()
TObjectPtr<UStaticMeshComponent> MeshComp;
```

它是 UE5 推荐的 UObject 指针封装形式，有利于对象引用追踪和编辑器工具链。

------

## 3.7 面试推荐回答

> UE 的 GC 主要管理 UObject 体系对象。GC 会从根对象出发，通过 UPROPERTY 等引用链查找仍然可达的对象，不可达的 UObject 会被回收。UObject 不应该手动 delete，Actor 也不应该手动 delete，而是调用 Destroy，由引擎处理生命周期。为了让 UObject 引用被 GC 正确识别，通常要用 UPROPERTY 保存引用。

------

# 4. NewObject 和 SpawnActor 的区别？

这是 UE C++ 面试非常高频的问题。

------

## 4.1 NewObject 是什么？

`NewObject` 用来创建 `UObject` 或其子类对象。

例如：

```cpp
UMyObject* Obj = NewObject<UMyObject>(this);
```

适合创建：

- UObject
- UDataAsset 的运行时对象
- 自定义数据对象
- 非场景对象
- 一些逻辑管理对象

例如：

```cpp
UInventoryItem* Item = NewObject<UInventoryItem>(this);
```

这个对象不是关卡中的 Actor，没有 Transform，也不会出现在 World 里。

------

## 4.2 SpawnActor 是什么？

`SpawnActor` 用来在 `World` 中生成 `Actor`。

例如：

```cpp
AEnemy* Enemy = GetWorld()->SpawnActor<AEnemy>(
    EnemyClass,
    SpawnLocation,
    SpawnRotation
);
```

适合生成：

- 敌人
- 子弹
- 武器
- 掉落物
- 特效 Actor
- 触发器
- 可交互物体

SpawnActor 生成的对象：

- 存在于 World
- 有 Transform
- 可以放入关卡
- 可以 Tick
- 可以参与碰撞
- 可以网络复制
- 可以被 World 管理生命周期

------

## 4.3 两者核心区别

| 对比             | NewObject      | SpawnActor           |
| ---------------- | -------------- | -------------------- |
| 创建对象类型     | UObject        | AActor               |
| 是否进入 World   | 否             | 是                   |
| 是否有 Transform | 通常没有       | 有                   |
| 是否能放到关卡   | 否             | 是                   |
| 是否用于场景对象 | 否             | 是                   |
| 生命周期管理     | Outer / GC     | World / Destroy / GC |
| 常见用途         | 数据、逻辑对象 | 敌人、子弹、道具     |

------

## 4.4 错误示例

不要用 `NewObject` 创建 Actor：

```cpp
AEnemy* Enemy = NewObject<AEnemy>();
```

这种写法不适合，因为 Actor 需要注册到 World 中，应该使用：

```cpp
AEnemy* Enemy = GetWorld()->SpawnActor<AEnemy>(EnemyClass);
```

也不要用 `SpawnActor` 创建普通 UObject：

```cpp
UMyObject* Obj = GetWorld()->SpawnActor<UMyObject>();
```

这是不对的，因为 `UMyObject` 不是 Actor。

------

## 4.5 面试推荐回答

> NewObject 用来创建 UObject 或其子类对象，这类对象通常不是场景中的实体，没有 Transform，不存在于 World 中。SpawnActor 用来在 World 中生成 Actor，生成的对象有 Transform，可以参与碰撞、Tick、网络复制，并由 World 管理生命周期。简单说，创建 UObject 用 NewObject，创建场景中的 Actor 用 SpawnActor。

------

# 5. CreateDefaultSubobject 是什么？

这个问题也很常见，尤其是写 C++ Actor 时。

------

## 5.1 CreateDefaultSubobject 用来做什么？

`CreateDefaultSubobject` 通常在构造函数中创建组件。

例如：

```cpp
AMyCharacter::AMyCharacter()
{
    CameraBoom = CreateDefaultSubobject<USpringArmComponent>(TEXT("CameraBoom"));
    CameraBoom->SetupAttachment(RootComponent);

    FollowCamera = CreateDefaultSubobject<UCameraComponent>(TEXT("FollowCamera"));
    FollowCamera->SetupAttachment(CameraBoom);
}
```

它创建的是 Actor 的默认子对象，也就是组件模板的一部分。

------

## 5.2 它和 NewObject 的区别

| 对比                 | CreateDefaultSubobject | NewObject            |
| -------------------- | ---------------------- | -------------------- |
| 使用位置             | 构造函数中             | 运行时也可用         |
| 常见用途             | 创建默认组件           | 创建 UObject         |
| 是否成为默认子对象   | 是                     | 否                   |
| 是否显示在蓝图组件树 | 通常可以               | 通常不显示为默认组件 |
| 生命周期             | 跟随 Owner             | 由 Outer / GC 管理   |

------

## 5.3 常见写法

```cpp
UCLASS()
class AMyActor : public AActor
{
    GENERATED_BODY()

public:
    AMyActor();

protected:
    UPROPERTY(VisibleAnywhere)
    UStaticMeshComponent* MeshComp;
};
AMyActor::AMyActor()
{
    MeshComp = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("MeshComp"));
    RootComponent = MeshComp;
}
```

------

## 5.4 面试推荐回答

> CreateDefaultSubobject 通常在构造函数中使用，用来创建 Actor 或组件类的默认子对象，比如 Mesh、Camera、Collision 等组件。它创建的组件会成为类默认对象的一部分，并能在蓝图组件树中显示。运行时动态创建 UObject 通常用 NewObject，生成 Actor 用 SpawnActor。

------

# 6. Constructor、BeginPlay、Tick 在 C++ 中有什么区别？

------

## 6.1 构造函数 Constructor

构造函数在对象创建时执行。

常见用途：

- 创建默认组件
- 设置默认属性
- 设置 Tick 是否开启
- 初始化基础数值

例如：

```cpp
AMyActor::AMyActor()
{
    PrimaryActorTick.bCanEverTick = true;

    MeshComp = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("MeshComp"));
    RootComponent = MeshComp;

    Health = 100.0f;
}
```

注意：

> 构造函数中不要依赖运行时世界状态，比如不要随便 GetWorld 后查找其他 Actor。

------

## 6.2 BeginPlay

`BeginPlay` 在游戏开始或 Actor 生成进入世界后调用。

常见用途：

- 获取其他对象引用
- 绑定事件
- 开启 Timer
- 初始化运行时逻辑
- 访问 GameMode、PlayerController 等运行时对象

```cpp
void AMyActor::BeginPlay()
{
    Super::BeginPlay();

    PlayerRef = UGameplayStatics::GetPlayerCharacter(this, 0);
}
```

------

## 6.3 Tick

`Tick` 每帧调用。

```cpp
void AMyActor::Tick(float DeltaTime)
{
    Super::Tick(DeltaTime);

    AddActorWorldOffset(Direction * Speed * DeltaTime);
}
```

常见用途：

- 每帧移动
- 插值
- 持续检测
- 摄像机跟随

但不要滥用 Tick。

------

## 6.4 面试推荐回答

> 构造函数主要用于创建默认组件和设置默认属性，执行时对象还不一定处于完整的游戏运行环境。BeginPlay 在 Actor 进入游戏世界后执行，适合获取运行时引用和初始化游戏逻辑。Tick 是每帧调用，适合连续更新逻辑，但要避免滥用。

------

# 7. UE C++ 中常见命名前缀有哪些？

UE 有一套比较固定的命名规范，面试也可能问。

------

## 7.1 类名前缀

| 前缀 | 含义              | 示例                                |
| ---- | ----------------- | ----------------------------------- |
| A    | Actor 类          | `ACharacter`, `APlayerController`   |
| U    | UObject 类        | `UActorComponent`, `UUserWidget`    |
| F    | Struct / 普通类型 | `FVector`, `FRotator`, `FHitResult` |
| I    | Interface         | `IInteractableInterface`            |
| E    | Enum              | `EWeaponType`                       |
| T    | Template          | `TArray`, `TMap`, `TSubclassOf`     |
| S    | Slate 控件        | `SButton`, `STextBlock`             |

------

## 7.2 变量命名

布尔变量通常使用 `b` 前缀：

```cpp
bool bIsDead;
bool bCanAttack;
bool bIsSprinting;
```

类成员变量常见写法：

```cpp
UPROPERTY()
float Health;

UPROPERTY()
int32 AmmoCount;

UPROPERTY()
AActor* TargetActor;
```

------

## 7.3 类型示例

```cpp
AActor* Actor;
ACharacter* Character;
UObject* Object;
UUserWidget* Widget;
FVector Location;
FRotator Rotation;
FHitResult HitResult;
TArray<AActor*> Actors;
TMap<FName, int32> ScoreMap;
```

------

## 7.4 面试推荐回答

> UE 中常见命名前缀有：A 表示 Actor 类，U 表示 UObject 类，F 表示结构体或普通类型，I 表示接口，E 表示枚举，T 表示模板类型，S 表示 Slate 控件，bool 变量通常用 b 开头，比如 bIsDead。

------

# 8. TSubclassOf 是什么？

这个问题在 UE C++ 中也很常见。

------

## 8.1 TSubclassOf 的作用

`TSubclassOf` 用来保存“某个类的子类类型”。

例如：

```cpp
UPROPERTY(EditAnywhere)
TSubclassOf<AActor> ProjectileClass;
```

这表示 `ProjectileClass` 可以在编辑器中指定一个 `AActor` 子类，比如：

- `BP_Bullet`
- `BP_Rocket`
- `BP_Fireball`

然后运行时可以生成它：

```cpp
GetWorld()->SpawnActor<AActor>(ProjectileClass, Location, Rotation);
```

------

## 8.2 为什么不用 AActor*？

因为：

```cpp
AActor* Projectile;
```

表示一个已经存在的 Actor 对象引用。

而：

```cpp
TSubclassOf<AActor> ProjectileClass;
```

表示一个类，也就是“以后要生成哪种 Actor”。

区别：

| 类型                  | 含义                |
| --------------------- | ------------------- |
| `AActor*`             | 一个 Actor 实例     |
| `TSubclassOf<AActor>` | 一个 Actor 类类型   |
| `UClass*`             | 通用类类型          |
| `TSubclassOf`         | 带类型限制的 UClass |

------

## 8.3 常见用途

```cpp
UPROPERTY(EditAnywhere)
TSubclassOf<AActor> EnemyClass;

UPROPERTY(EditAnywhere)
TSubclassOf<UUserWidget> HealthBarWidgetClass;

UPROPERTY(EditAnywhere)
TSubclassOf<UGameplayAbility> DefaultAbilityClass;
```

------

## 8.4 面试推荐回答

> TSubclassOf 用来保存某个类的子类类型，常用于在编辑器中指定要生成的蓝图类。比如 TSubclassOf ProjectileClass 可以指定一个子弹蓝图类，然后用 SpawnActor 生成。AActor* 表示实例，TSubclassOf 表示类。

------

# 9. Cast 和 CastChecked 在 C++ 中有什么区别？

------

## 9.1 Cast

UE 中常用：

```cpp
AMyCharacter* MyChar = Cast<AMyCharacter>(OtherActor);
```

如果转换成功，返回目标类型指针。

如果失败，返回 `nullptr`。

所以通常要判断：

```cpp
if (AMyCharacter* MyChar = Cast<AMyCharacter>(OtherActor))
{
    MyChar->Attack();
}
```

------

## 9.2 CastChecked

```cpp
AMyCharacter* MyChar = CastChecked<AMyCharacter>(OtherActor);
```

如果转换失败，会触发断言或崩溃。

适合你非常确定类型一定正确的情况。

------

## 9.3 面试推荐回答

> Cast 是安全类型转换，失败返回 nullptr。CastChecked 则表示我确信它一定能转换成功，如果失败会触发断言或崩溃。所以普通逻辑中更常用 Cast，只有非常确定类型时才使用 CastChecked。

------

# 10. UE C++ 中如何暴露变量给编辑器和蓝图？

------

## 10.1 暴露给编辑器

```cpp
UPROPERTY(EditAnywhere)
float Damage;
```

`EditAnywhere` 表示可以在类默认值和实例中编辑。

常见还有：

```cpp
UPROPERTY(EditDefaultsOnly)
float Damage;
```

只允许在蓝图类默认值中编辑。

```cpp
UPROPERTY(EditInstanceOnly)
float PatrolRadius;
```

只允许在关卡实例中编辑。

------

## 10.2 暴露给蓝图读取或写入

```cpp
UPROPERTY(BlueprintReadOnly)
float Health;
```

蓝图可读。

```cpp
UPROPERTY(BlueprintReadWrite)
float Health;
```

蓝图可读可写。

------

## 10.3 常见组合

```cpp
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Combat")
float Damage = 20.0f;
```

表示：

- 编辑器可编辑
- 蓝图可读写
- 显示在 Combat 分类下

------

## 10.4 对组件常用 VisibleAnywhere

组件通常写：

```cpp
UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category="Components")
UStaticMeshComponent* MeshComp;
```

原因是组件本身一般不希望被随意替换，但希望能在编辑器中看到。

------

## 10.5 面试推荐回答

> UE C++ 中用 UPROPERTY 暴露变量。EditAnywhere、EditDefaultsOnly、EditInstanceOnly 控制编辑器可编辑范围；BlueprintReadOnly 和 BlueprintReadWrite 控制蓝图访问权限；Category 控制在编辑器中的分类。组件一般用 VisibleAnywhere，因为希望可见但不一定希望被随意替换。

------

# 11. UFUNCTION 常见修饰符有哪些？

------

## 11.1 BlueprintCallable

```cpp
UFUNCTION(BlueprintCallable)
void Attack();
```

蓝图可以调用这个函数，有执行引脚。

适合：

- 攻击
- 受伤
- 开门
- 初始化
- 使用道具

------

## 11.2 BlueprintPure

```cpp
UFUNCTION(BlueprintPure)
float GetHealthPercent() const;
```

蓝图纯函数，没有执行引脚。

适合：

- 获取血量百分比
- 判断是否死亡
- 返回某个计算结果

------

## 11.3 BlueprintImplementableEvent

```cpp
UFUNCTION(BlueprintImplementableEvent)
void OnAttackHit();
```

C++ 声明事件，蓝图实现。

C++ 可以调用它：

```cpp
OnAttackHit();
```

然后实际表现逻辑在蓝图中实现。

适合：

- 播放特效
- 播放音效
- UI反馈
- 动画表现

------

## 11.4 BlueprintNativeEvent

```cpp
UFUNCTION(BlueprintNativeEvent)
void Interact();
```

C++ 可以提供默认实现，蓝图也可以重写。

实现时写：

```cpp
void AMyActor::Interact_Implementation()
{
    // 默认交互逻辑
}
```

------

## 11.5 面试推荐回答

> UFUNCTION 可以让函数被 UE 反射系统识别。BlueprintCallable 表示蓝图可以调用，BlueprintPure 表示蓝图纯函数，BlueprintImplementableEvent 表示函数由蓝图实现，BlueprintNativeEvent 表示 C++ 有默认实现但蓝图可以重写。网络 RPC 也需要通过 UFUNCTION 标记。

------

# 12. UObject、Actor、Component 生命周期区别

------

## 12.1 UObject

UObject 是 UE 对象系统的基础。

特点：

- 由 GC 管理
- 不能直接放入场景
- 通常没有 Transform
- 适合作为数据对象、逻辑对象、资源对象

创建方式：

```cpp
UMyObject* Obj = NewObject<UMyObject>(this);
```

------

## 12.2 Actor

Actor 是能存在于 World 中的对象。

特点：

- 有 Transform
- 可以放到关卡里
- 可以 SpawnActor
- 可以 Tick
- 可以参与碰撞
- 可以网络复制

创建方式：

```cpp
AEnemy* Enemy = GetWorld()->SpawnActor<AEnemy>(EnemyClass);
```

销毁方式：

```cpp
Enemy->Destroy();
```

------

## 12.3 Component

Component 是挂在 Actor 上的功能模块。

常见：

- MeshComponent
- CollisionComponent
- CameraComponent
- MovementComponent
- AudioComponent

默认组件一般在构造函数中创建：

```cpp
MeshComp = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("MeshComp"));
```

运行时动态组件可以用 `NewObject` 创建，并注册：

```cpp
UStaticMeshComponent* NewComp = NewObject<UStaticMeshComponent>(this);
NewComp->RegisterComponent();
NewComp->AttachToComponent(RootComponent, FAttachmentTransformRules::KeepRelativeTransform);
```

------

## 12.4 面试推荐回答

> UObject 是 UE 对象系统基础，主要由 GC 管理，不能直接放入场景。Actor 是可以存在于 World 中的对象，有 Transform，可以 SpawnActor 生成，用 Destroy 销毁。Component 是挂在 Actor 上的功能模块，默认组件通常在构造函数中用 CreateDefaultSubobject 创建。

------

# 13. Super 是什么？为什么要调用 Super？

你之前也问过 `Super`，这个也经常被问。

------

## 13.1 Super 是什么？

在 UE 中，`Super` 表示当前类的父类。

例如：

```cpp
class AMyCharacter : public ACharacter
```

在 `AMyCharacter` 中：

```cpp
Super
```

就表示 `ACharacter`。

------

## 13.2 为什么要调用 Super？

因为父类中可能已经实现了重要逻辑。

例如：

```cpp
void AMyCharacter::BeginPlay()
{
    Super::BeginPlay();

    // 自己的初始化逻辑
}
```

`Super::BeginPlay()` 可能会执行父类的初始化逻辑。如果不调用，可能导致父类功能不完整。

------

## 13.3 常见需要调用 Super 的函数

```cpp
BeginPlay()
Tick()
SetupPlayerInputComponent()
PossessedBy()
OnRep_PlayerState()
EndPlay()
```

例如：

```cpp
void AMyCharacter::Tick(float DeltaTime)
{
    Super::Tick(DeltaTime);

    // 自己的每帧逻辑
}
```

------

## 13.4 面试推荐回答

> Super 表示当前类的父类。调用 Super::Function 是为了保留父类已经实现的基础逻辑，比如初始化、Tick、输入绑定、生命周期处理等。如果重写函数但不调用 Super，可能导致父类逻辑没有执行，引发异常行为。

------

# 14. check、ensure、UE_LOG 是什么？

这些是 UE C++ 调试常见内容。

------

## 14.1 UE_LOG

`UE_LOG` 用于打印日志。

```cpp
UE_LOG(LogTemp, Warning, TEXT("Health: %f"), Health);
```

常见级别：

```cpp
UE_LOG(LogTemp, Log, TEXT("Normal Log"));
UE_LOG(LogTemp, Warning, TEXT("Warning Log"));
UE_LOG(LogTemp, Error, TEXT("Error Log"));
```

------

## 14.2 check

`check` 类似断言。

```cpp
check(PlayerController);
```

如果条件为 false，程序会直接中断。

适合检查理论上绝对不应该失败的条件。

------

## 14.3 ensure

`ensure` 也会检查条件，但比 `check` 温和。

```cpp
ensure(PlayerController);
```

如果失败，会报告错误，但通常不会直接让程序崩溃，适合调试非致命问题。

------

## 14.4 面试推荐回答

> UE_LOG 用于输出日志。check 是强断言，条件失败通常会中断程序，适合检查绝不应该发生的问题。ensure 是较温和的断言，失败时会报告问题但通常不立即崩溃，适合调试可恢复的异常情况。

------

# 15. 第三部分综合面试模板

如果面试官问：“你怎么理解 UE C++？”

可以这样回答：

> UE C++ 在标准 C++ 基础上加入了反射系统，核心宏包括 UCLASS、USTRUCT、UFUNCTION、UPROPERTY。它们让 C++ 类、结构体、函数和变量能被编辑器、蓝图、序列化、网络复制和 GC 系统识别。
>
> UPROPERTY 很重要，尤其是 UObject 指针需要通过 UPROPERTY 或 TObjectPtr 保存，这样 GC 才能正确追踪引用，避免对象被错误回收。UE 的 GC 主要管理 UObject 体系对象，普通 UObject 不手动 delete，Actor 一般通过 Destroy 销毁。
>
> 创建对象时，UObject 用 NewObject，Actor 用 SpawnActor，默认组件通常在构造函数中用 CreateDefaultSubobject 创建。构造函数适合设置默认值和组件，BeginPlay 适合运行时初始化，Tick 适合连续更新但要避免滥用。
>
> UE 还有自己的命名规范，比如 A 表示 Actor，U 表示 UObject，F 表示结构体，I 表示接口，E 表示枚举，T 表示模板类型，bool 变量用 b 前缀。

------

# 16. 简短版背诵答案

面试时可以背这个版本：

> UE C++ 的核心特点是反射系统。UCLASS、USTRUCT、UFUNCTION、UPROPERTY 这些宏可以让 C++ 类型、函数和变量被引擎识别，从而支持蓝图、编辑器、序列化、网络复制和垃圾回收。
>
> UPROPERTY 对 UObject 指针很重要，因为 UE 的 GC 通过反射系统追踪引用关系。如果 UObject 指针没有 UPROPERTY 标记，可能会被错误回收。
>
> UObject 通常用 NewObject 创建，由 GC 管理；Actor 用 SpawnActor 生成到 World 中，用 Destroy 销毁；组件通常在构造函数中用 CreateDefaultSubobject 创建。
>
> 构造函数用于设置默认值和创建默认组件，BeginPlay 用于运行时初始化，Tick 每帧调用但不要滥用。UE 命名中 A 表示 Actor，U 表示 UObject，F 表示结构体，I 表示接口，E 表示枚举，T 表示模板，bool 变量用 b 开头。