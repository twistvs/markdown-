#### UCLASS()与UClass的区别

UCLASS()是宏声明，可以将UObject类加入反射系统

UClass是一个类，UClass对象保存了某个类的相关信息

> [!NOTE]
>
> 假如说我们有一个类
>
> ```c++
> UCLASS()
> class UMyObject : public UObject{
>     
>     GENERATED_BODY()
>         
> private:
>     
>     UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "AbilityInfo")
>     UMaterialInstance* IconMaterial;
>     
> public:
> 
> }
> ```
>
> 我们知道UMyObject是一个类，而他可以创建自己的对象，他的对象包含了IconMaterial这个成员，**类的对象记录的是成员的信息**
>
> 而UClass不是，**UClass对象记录的是某个类自己的信息，比如：类的名字、父类是谁、类的函数是啥，类的默认值CDO是啥**

​	Class是一个UClass对象，他记录了另一个类的信息，用Class->GetName()就可以获得这个类的名字，可以看到，**UClass记录的是类本身的信息，而这些信息在平时经常被我们给忽略掉**

```c++
UClass* Class = Object->GetClass();

FString Name = Class->GetName();
UClass* Parent = Class->GetSuperClass();
```

------

#### UObject值成员不能放在UCLASS()和USTRUCT()内

```c++
USTRUCT()
struct FMyData
{
    GENERATED_BODY()

    UObject Object;     // 不支持
    UMyObject MyObject; // 不支持，假设 UMyObject 继承 UObject
};
```

只要不是UObject，就可以放，比如FMyStruct
