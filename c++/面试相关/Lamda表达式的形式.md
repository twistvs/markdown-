C++ 中的 Lambda 表达式（Lambda Expression） 是一种匿名函数，可以在代码中直接定义函数逻辑，常用于 STL 算法、回调函数和异步任务。

## 一、Lambda 表达式的基本形式

```
[capture](parameters) mutable -> return_type
{
    // 函数体
};
```

更完整的 C++11 常见形式是：

```
[捕获列表](参数列表) mutable -> 返回值类型
{
    函数体
};
```

其中，`mutable` 和 `-> 返回值类型` 都可以省略。

| 组成部分         | 作用                   | 是否可以省略                 |
| ---------------- | ---------------------- | ---------------------------- |
| `[capture]`      | 捕获外部变量           | 方括号不能省略，内容可以为空 |
| `(parameters)`   | 函数参数列表           | 无参数时通常可以省略         |
| `mutable`        | 允许修改按值捕获的副本 | 可以                         |
| `-> return_type` | 指定返回值类型         | 可以，通常自动推导           |
| `{}`             | Lambda 函数体          | 不可以                       |

最简单的 Lambda：

```
auto func = []()
{
    std::cout << "Hello Lambda" << std::endl;
};

func(); // 调用 Lambda
```

还可以写成：

```
auto add = [](int a, int b) -> int
{
    return a + b;
};

std::cout << add(3, 5); // 8
```

因为编译器能够自动推导返回类型，所以可以简化为：

```
auto add = [](int a, int b)
{
    return a + b;
};
```

## 二、捕获列表的几种形式

Lambda 与普通函数最大的区别之一，就是可以捕获所在作用域中的变量。

```
int a = 10;
int b = 20;
```

| 捕获形式  | 含义                         |
| --------- | ---------------------------- |
| `[]`      | 不捕获任何外部变量           |
| `[=]`     | 按值捕获用到的外部局部变量   |
| `[&]`     | 按引用捕获用到的外部局部变量 |
| `[a]`     | 按值捕获变量 `a`             |
| `[&a]`    | 按引用捕获变量 `a`           |
| `[a, &b]` | `a` 按值捕获，`b` 按引用捕获 |
| `[=, &a]` | 默认按值捕获，`a` 按引用捕获 |
| `[&, a]`  | 默认按引用捕获，`a` 按值捕获 |
| `[this]`  | 捕获当前对象的 `this` 指针   |
| `[*this]` | 按值捕获当前对象（C++17 起） |

### 1. 值捕获

```
int a = 10;

auto func = [a]()
{
    std::cout << a;
};

a = 20;

func(); // 10
```

Lambda 保存的是捕获时 `a` 的值，因此外部变量变化不会影响捕获的副本。

### 2. 引用捕获

```
int a = 10;

auto func = [&a]()
{
    a = 30;
};

func();

std::cout << a; // 30
```

引用捕获可以修改外部变量。

注意：如果 Lambda 的生命周期超过被引用变量的生命周期，后续访问就可能产生悬空引用。

### 3. mutable 的作用

默认情况下，Lambda 的调用运算符通常是 `const`，因此不能直接修改按值捕获的普通变量。

```
int a = 10;

auto func = [a]() mutable
{
    a++;
    std::cout << a;
};

func(); // 11

std::cout << a; // 10
```

这里的 `mutable` 允许修改 Lambda 内部的副本，不会修改外部的 `a`。

## 三、Lambda 表达式的底层原理

Lambda 本质上是编译器生成的匿名闭包类（Closure Type）的对象。

例如：

```
int a = 10;

auto func = [a](int b)
{
    return a + b;
};
```

可以近似理解成编译器生成了：

```
class Lambda
{
private:
    int a;

public:
    Lambda(int value) : a(value) {}

    int operator()(int b) const
    {
        return a + b;
    }
};
```

然后：

```
Lambda func(a);

func(20); // 30
```

所以 Lambda 为什么可以像函数一样调用？

因为编译器生成的闭包类重载了：

```
operator()
```

这也说明 Lambda 本质上是一种函数对象（Functor）。

## 四、常见使用场景

### 1. STL 自定义排序

```
std::vector<int> nums = {3, 1, 5, 2, 4};

std::sort(nums.begin(), nums.end(),
    [](int a, int b)
    {
        return a > b;
    });
```

这里的 Lambda 充当比较函数，实现降序排序。

### 2. 遍历容器

```
std::vector<int> nums = {1, 2, 3, 4, 5};

std::for_each(nums.begin(), nums.end(),
    [](int x)
    {
        std::cout << x << " ";
    });
```

### 3. 泛型 Lambda（C++14）

```
auto add = [](auto a, auto b)
{
    return a + b;
};

add(1, 2);       // int
add(1.5, 2.5);   // double
```

### 4. 立即调用 Lambda

```
int result = [](int a, int b)
{
    return a + b;
}(3, 5);

std::cout << result; // 8
```

这种写法定义 Lambda 后立即调用。

## 五、面试总结

面试官问：“C++ Lambda 表达式的形式是什么？”

可以回答：

> Lambda 表达式的基本形式是 `[捕获列表](参数列表) -> 返回类型 {函数体}`，还可以使用 `mutable` 等说明符。
>
> 其中捕获列表可以采用值捕获、引用捕获或混合捕获。值捕获通常保存变量的副本，引用捕获允许访问原变量。
>
> Lambda 的底层原理是编译器生成一个匿名闭包类，并重载 `operator()`，因此 Lambda 本质上是一个函数对象。
>
> Lambda 经常用于 STL 算法、自定义排序、回调函数和异步任务。

重点记住：`[]` 是捕获列表，`()` 是参数列表，`{}` 是函数体。 其中捕获列表是 Lambda 区别于普通函数的重要特征。