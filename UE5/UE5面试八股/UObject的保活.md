在 UE5 中，保活 UObject 的核心原理是让对象始终能够被垃圾回收系统（GC）从根集合或其他可达对象找到，从而避免被回收。

从 UE5 面试的角度来看，常见的保活手段主要有 5 种：

1. `UPROPERTY()` + `TObjectPtr`（最常用）
2. `AddToRoot()`（加入根集合）
3. `TStrongObjectPtr`（强引用智能指针）
4. `FGCObject`（普通 C++ 类参与 GC 引用收集）
5. `AddReferencedObjects()`（自定义 GC 引用收集）

## 一、五种保活手段详解

### 1. UPROPERTY + TObjectPtr

这是最常用、最推荐的方式。

```c++
UCLASS()
class UMyManager : public UObject
{
    GENERATED_BODY()

public:
    UPROPERTY()
    TObjectPtr<UObject> MyObject;
};
```

当 `UMyManager` 被 GC 判定为可达时，GC 会沿着 `MyObject` 这个强引用继续寻找对象。

因此，只要 `UMyManager` 存活，并且引用关系没有被清除，`MyObject` 指向的对象通常就不会被 GC 回收。

需要注意：

- `UPROPERTY()` 使成员引用能够被 UE 的 GC 系统追踪。
- `TObjectPtr` 本身不等于永久保活，通常需要配合 `UPROPERTY()` 或其他 GC 引用收集机制。
- 如果持有引用的 `UMyManager` 自身不可达，那么仅靠这个成员变量并不能保活目标对象。

UE5 仍然支持使用 `UPROPERTY()` 标记原始 `UObject*` 作为强引用，但更推荐新代码使用 `TObjectPtr`。

### 2. AddToRoot

直接将 UObject 加入 GC 根集合，使它不被正常 GC 回收。

```c++
UObject* Obj = NewObject<UObject>();

Obj->AddToRoot();
```

不再需要时：

```c++
Obj->RemoveFromRoot();
```

原理是：GC 以根集合为起点遍历对象引用关系。被加入根集合的对象会直接成为 GC 的根对象。

优点是保活效果直接，不依赖其他 UObject 持有引用。

缺点是必须正确管理 `RemoveFromRoot()`，否则容易造成对象长期无法回收。

面试注意：`AddToRoot()` 只能阻止正常 GC 回收，不能保证 Actor 不会被 `Destroy()` 销毁，也不能阻止引擎其他生命周期管理逻辑。

### 3. TStrongObjectPtr

`TStrongObjectPtr` 是 UE 提供的 UObject 强引用指针，可以防止其指向的 UObject 被 GC 回收。

```c++
TStrongObjectPtr<UObject> Ptr(
    NewObject<UObject>()
);
```

它的优势是利用 RAII 自动管理强引用的建立与释放。

当 `TStrongObjectPtr` 被销毁或重置时，对目标 UObject 的这份保活引用也会被释放。

它特别适用于普通 C++ 类或局部作用域中需要持有 UObject 的场景。

需要注意，它与 `TSharedPtr` 不一样：

- `TSharedPtr` 使用共享引用计数管理普通 C++ 对象。
- `TStrongObjectPtr` 是为 UObject 的 GC 体系提供强引用，不通过共享引用计数决定 UObject 的销毁。

### 4. FGCObject

如果一个普通 C++ 类并不继承 `UObject`，但希望自己持有的 UObject 引用能够被 GC 识别，可以继承 `FGCObject`。

```c++
class FMyManager : public FGCObject
{
public:
    UObject* MyObject = nullptr;

    virtual void AddReferencedObjects(
        FReferenceCollector& Collector
    ) override
    {
        Collector.AddReferencedObject(MyObject);
    }

    virtual FString GetReferencerName() const override
    {
        return TEXT("FMyManager");
    }
};
```

`FGCObject` 会将这个管理类注册到 GC 引用收集体系中。

当 GC 执行时，会调用 `AddReferencedObjects`，把 `MyObject` 报告给引用收集器。

这样，即使 `FMyManager` 不是 UObject，也能保活自己持有的 UObject。

常见使用场景是非 UObject 的管理器、编辑器工具以及其他普通 C++ 类。

### 5. 自定义 AddReferencedObjects

对于继承自 UObject 的类，如果某些引用无法通过普通 `UPROPERTY` 反射机制追踪，可以通过自定义 `AddReferencedObjects()` 向 GC 报告引用。

示意代码：

```c++
UCLASS()
class UMyManager : public UObject
{
    GENERATED_BODY()

public:
    UObject* MyObject = nullptr;

    static void AddReferencedObjects(
        UObject* InThis,
        FReferenceCollector& Collector
    );
};
```

对应实现：

```c++
void UMyManager::AddReferencedObjects(
    UObject* InThis,
    FReferenceCollector& Collector)
{
    UMyManager* This = CastChecked<UMyManager>(InThis);

    Super::AddReferencedObjects(InThis, Collector);

    Collector.AddReferencedObject(This->MyObject);
}
```

这样，GC 在扫描该管理对象时，也能发现 `MyObject` 这条引用。

和 `FGCObject` 的区别在于：

- `FGCObject` 主要让普通 C++ 类参与 GC 引用收集。
- `UObject::AddReferencedObjects` 用于 UObject 自定义引用收集。

## 二、哪些指针不能保活 UObject？

这是面试中非常容易考察的细节。

| 指针类型                    | 能否保活 UObject              |
| --------------------------- | ----------------------------- |
| `UPROPERTY() TObjectPtr<T>` | 能，前提是持有者可达          |
| `UPROPERTY() UObject*`      | 能，前提是持有者可达          |
| `TStrongObjectPtr<T>`       | 能                            |
| `TWeakObjectPtr<T>`         | 不能                          |
| `TSoftObjectPtr<T>`         | 不能单独保活                  |
| 普通 `UObject*` 局部变量    | 不保证保活                    |
| `TSharedPtr<T>`             | 不适用于 UObject 生命周期管理 |

另外，`Outer` 关系本身并不保证 Outer 会保活 Inner 对象。这是一个很容易答错的知识点。

例如：

```
UObject* Obj = NewObject<UObject>(GetGameInstance());
```

虽然把 `GameInstance` 设置为了 `Outer`，但不能因此认为 `Obj` 必然受到 GC 保护。如果需要长期保存它，应建立正确的强引用关系。

## 三、不同使用场景应该怎么选择？

| 场景                                    | 推荐方式                 |
| --------------------------------------- | ------------------------ |
| Actor 或 UObject 成员保存其他 UObject   | `UPROPERTY() TObjectPtr` |
| 普通 C++ 类需要长期管理多个 UObject     | `FGCObject`              |
| 普通 C++ 代码需要方便地保活单个 UObject | `TStrongObjectPtr`       |
| 对象需要明确加入 GC 根集合              | `AddToRoot`              |
| 对象引用无法使用 UPROPERTY 反射         | `AddReferencedObjects`   |
| 只需要观察 UObject，不希望阻止 GC       | `TWeakObjectPtr`         |

## 四、面试推荐回答

> UE5 主要有五种保活 UObject 的手段。
>
> 第一种是通过 `UPROPERTY` 配合 `TObjectPtr` 建立强引用，这是最常见的方式，只要持有者可达，GC 就能够追踪到被引用对象。
>
> 第二种是调用 `AddToRoot`，直接把对象加入 GC 根集合，阻止正常垃圾回收，需要手动调用 `RemoveFromRoot`。
>
> 第三种是 `TStrongObjectPtr`，它可以通过 RAII 机制自动管理 UObject 的强引用。
>
> 第四种是继承 `FGCObject`，主要用于普通 C++ 类，让它们也能够向 GC 报告 UObject 引用。
>
> 第五种是自定义 `AddReferencedObjects`，用于手动向 GC 收集器报告无法通过反射机制追踪的引用。
>
> 其中，实际开发中最常用的是 `UPROPERTY` 配合 `TObjectPtr`，其他方式则根据对象的持有关系和生命周期需求选择。

最后有一个关键点：UObject 的保活不是简单地让某个指针不为 `nullptr`，而是确保 GC 能从根集合通过有效的强引用链找到这个对象。