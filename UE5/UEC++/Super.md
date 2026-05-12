在 UE C++ 里，`Super` 可以理解为：

> **当前类的父类。**

它通常用来调用父类中已经实现好的函数，避免你重写函数后把父类原本的逻辑覆盖掉。

在一般情况下，在子类中重写父类的函数，会导致从父类继承来的函数被直接覆盖掉，子类无法执行父类的函数，而通过Super，我们可以无视覆盖来调用父类的函数

------

## 1. 最常见用法

比如你的角色类继承自 `ACharacter`：

```cpp
class GAS_DEMO_API AMyCharacter : public ACharacter
{
    GENERATED_BODY()
};
```

那么在 `AMyCharacter` 里：

```cpp
Super
```

就代表：

```cpp
ACharacter
```

如果你的类继承自 `AActor`：

```cpp
class GAS_DEMO_API AMyActor : public AActor
{
    GENERATED_BODY()
};
```

那么：

```cpp
Super
```

就代表：

```cpp
AActor
```

------

## 2. 为什么 BeginPlay 里经常写 Super::BeginPlay()

例如：

```cpp
void AMyCharacter::BeginPlay()
{
    Super::BeginPlay();

    // 你自己的初始化逻辑
}
```

意思是：

```text
先执行父类的 BeginPlay
再执行我自己的 BeginPlay 逻辑
```

如果你不写：

```cpp
Super::BeginPlay();
```

父类的 `BeginPlay` 逻辑就不会执行。

对于 UE 的 Actor、Character、Component、AttributeSet 等类来说，父类里可能有很多引擎内部初始化逻辑，所以通常建议保留。

------

## 3. Super 和普通 C++ 父类调用的关系

在标准 C++ 里，调用父类函数通常这样写：

```cpp
ACharacter::BeginPlay();
```

UE 里用：

```cpp
Super::BeginPlay();
```

本质上类似。

`Super` 是 UE 的反射宏帮你定义的父类别名。

比如：

```cpp
UCLASS()
class GAS_DEMO_API AMyCharacter : public ACharacter
{
    GENERATED_BODY()
};
```

经过 UE 宏处理后，大致相当于给你提供了：

```cpp
using Super = ACharacter;
```

所以你可以写：

```cpp
Super::BeginPlay();
```

而不用直接写：

```cpp
ACharacter::BeginPlay();
```

------

## 4. 为什么推荐用 Super，而不是直接写父类名？

假设你现在的继承关系是：

```cpp
AMyCharacter : public ACharacter
```

你写：

```cpp
ACharacter::BeginPlay();
```

目前可以。

但以后你改成：

```cpp
AMyCharacter : public ABaseCharacter
```

如果你之前到处写的是：

```cpp
ACharacter::BeginPlay();
```

就需要全部手动改。

但如果你一直写：

```cpp
Super::BeginPlay();
```

UE 会自动代表当前父类，代码更容易维护。

------

## 5. 常见使用位置

### BeginPlay

```cpp
void AMyActor::BeginPlay()
{
    Super::BeginPlay();

    UE_LOG(LogTemp, Warning, TEXT("My BeginPlay"));
}
```

------

### Tick

```cpp
void AMyActor::Tick(float DeltaTime)
{
    Super::Tick(DeltaTime);

    // 每帧逻辑
}
```

------

### SetupPlayerInputComponent

```cpp
void AMyCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
    Super::SetupPlayerInputComponent(PlayerInputComponent);

    // 绑定输入
}
```

------

### 构造函数

构造函数里也经常看到：

```cpp
AMyCharacter::AMyCharacter()
{
    PrimaryActorTick.bCanEverTick = true;
}
```

有些类构造函数会显式调用父类构造函数：

```cpp
AMyCharacter::AMyCharacter(const FObjectInitializer& ObjectInitializer)
    : Super(ObjectInitializer)
{
}
```

这里的：

```cpp
: Super(ObjectInitializer)
```

意思是调用父类构造函数。

------

## 6. 不调用 Super 会怎样？

看具体函数。

有些函数不调用也许暂时没事，但很多 UE 生命周期函数不调用 `Super` 可能导致隐藏问题。

例如：

```cpp
void AMyActor::BeginPlay()
{
    // 没有 Super::BeginPlay()
}
```

可能导致父类或组件的一些初始化逻辑没有正常执行。

再比如：

```cpp
void AMyCharacter::Tick(float DeltaTime)
{
    // 没有 Super::Tick(DeltaTime)
}
```

有时可能影响父类内部 Tick 逻辑。

所以一般规则是：

```text
重写 UE 生命周期函数时，默认先调用 Super
```

例如：

```cpp
Super::BeginPlay();
Super::Tick(DeltaTime);
Super::SetupPlayerInputComponent(PlayerInputComponent);
Super::PossessedBy(NewController);
Super::OnRep_PlayerState();
```

------

## 7. 有没有不需要调用 Super 的情况？

有，但要看你是否明确想完全替换父类逻辑。

例如某些自定义函数，父类逻辑为空，或者你就是要阻止父类行为，这时可以不调用。

但在 UE 初学阶段，建议：

```text
除非你明确知道父类做了什么，并且确定不要它，否则保留 Super 调用
```

------

## 8. 在 GAS 里也常见

比如 AttributeSet 里你可能会重写：

```cpp
virtual void PreAttributeChange(
    const FGameplayAttribute& Attribute,
    float& NewValue
) override;
```

实现时一般写：

```cpp
void UBaseAttributeSet::PreAttributeChange(
    const FGameplayAttribute& Attribute,
    float& NewValue
)
{
    Super::PreAttributeChange(Attribute, NewValue);

    // 你自己的属性限制逻辑
}
```

意思是：

```text
先执行 UAttributeSet 父类原本的 PreAttributeChange
再执行自己的逻辑
```

------

## 9. 一句话总结

`Super` 就是：

```text
当前类的父类
```

比如：

```cpp
Super::BeginPlay();
```

意思是：

```text
调用父类的 BeginPlay()
```

最常见用途是：**在重写 UE 生命周期函数时，保留父类原本逻辑，然后再添加自己的逻辑。**