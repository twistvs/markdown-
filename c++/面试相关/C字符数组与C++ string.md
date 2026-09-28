#### 首先回顾一下各种数据类型的字节数


```
bool        1
char        1
short       2
int         4
long        4
long long   8
float       4
double      8
指针        8
```

------

#### C字符数组

```c++
char str[] = "abcdefg"
cout << sizeof(str) << endl;   //输出8
cout << strlen(str) << endl;   //输出7
```

**sizeof的原理**

​	sizeof并不关心str有什么内容，具体是什么样，他只关心str的**类型**

​	sizeof在**编译期**获取str的类型为char[8]（长度为8的字符数组，包括字符串结束标志'\0'），而char[8]的sizeof是8，所以就输出8。

**strlen的原理**

​	strlen接受一个字符指针`char*`，数组名本身的类型是`char[8]`，在传参时退化成指向第一个元素的指针`char*`，strlen函数从这个指针开始，依次寻找字符串结束标志'\0'，最终返回字符数组的真实长度（不包括'\0'）

```c++
size_t my_strlen(const char* str)
{
    size_t len = 0;

    while (*str != '\0')
    {
        ++len;
        ++str;
    }

    return len;
}
```

------

#### C++容器string

​	string本身是一个类，而类对象的内容肯定不只包含字符串"abcdefg"本身，还包括了大量的其他数据

```
std::string 对象
+------------------+
| 指针/内部缓冲区   |
| size             |
| capacity         |
| 其他实现数据      |
+------------------+
```

结果取决于你的编译器、标准库实现和平台，**不是固定值**。在常见的 64 位环境下，你可能看到：

```
sizeof(s) == 24
```

或者：

```
sizeof(s) == 32
```

都有可能。

------

