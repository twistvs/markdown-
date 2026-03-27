- const关键字的作用是**向编辑器和程序员承诺：某个值不会被修改**

------

#### const关键字修饰变量（全局或局部）

不会修改变量的作用域，只是让变量无法被修改

------

#### const关键字修饰指针

```c++
//常量指针，指向的内容不能被修改
int a;
const int* p = &a;

//指针常量，指针的指向不能被修改
int a;
int* const p = &a;
int b;
p = &b;//错误
```

------

#### const关键字修饰函数参数和返回值

```c++
//提高代码健壮性，为了避免拷贝提高效率而使用了引用，而引用就有可能修改vec的值，如果vec是不应该被修改的，为了同时保留引用的高效和避免误修改vec，可以用const关键字
void func(const vector<int>& vec){}

//提高代码健壮性，避免不应该被修改的返回值在之后被误修改
const int func(int a){}
```

------

#### const关键字修饰类成员（包括成员变量和成员函数）

- **const修饰成员变量**的作用和修饰变量是一样的**，但是const成员变量只能在初始化列表中进行初始化，不能在函数中存入值**

const变量在被创建时就必须要初始化并且不能被修改，初始化列表就是在const成员变量创建时进行初始化，而在函数中赋值就变成了先创建再赋值，是不可以的

```c++
class Person {
public:
	const int a;
	//用初始化列表可以给a进行初始化
	Person() :a(10) {}
    
    //这样也可以，在声明的时候直接初始化
    const int b = 10;
};
```

```c++
class Person {
public:
	const int a;
	
    //这样不行
	Person() {
		a = 10;
	}
};
```

------

**const修饰成员函数**会把this从指针常量变成常量指针常量，**只能修改静态成员变量**（因为修改静态成员变量根本不需要用到this，而const仅仅只是修改了this，所以自然也就不影响）

```c++
class Person{
public:
	int age = 10;
    
    //成员函数在调用成员变量的时候，会使用到this指针，而const成员函数的this指针是const Person* const，指向的内容不能被改变，所以this->age自然也就无法被修改
    void setAge(int age) const{
        this->age = age;
    }
}
```

```c++
# include <iostream>
class Person {
public:
	static int a;

	void setA(int val) const {
        //由于a不用this指针，所以不受const成员函数的约束
		a = val;
	}
};
//静态成员变量的类外定义
int Person::a = 10;

int main() {
	Person p1;
	p1.setA(20);  //成功用const成员函数修改了静态成员变量的值
	std::cout << Person::a << std::endl;
	
	return 0;
}
```

------

#### const对象

const对象的this会变成常量指针，也就是this指向的一切变量不能被改变，并且普通成员函数也无法再用这个this

| **特性**              | **普通对象**       | **const 对象**                         |
| --------------------- | ------------------ | -------------------------------------- |
| **修改成员变量**      | 允许               | **禁止**（除非变量标记为 `mutable`）   |
| **调用普通函数**      | 允许               | **禁止**                               |
| **调用 `const` 函数** | 允许               | **允许**                               |
| **传递方式**          | 常通过值或引用传递 | **必须**通过常量引用（`const T&`）传递 |

```c++
class Person{
public:
	void func(){}  //成员函数在被调用时，会使用this指针来找到属于对象p1的成员变量，而这个this指针是Person* const类型
    void func2() const {
		std::cout << "被调用了" << std::endl;
	}
}

int main(){
    const Person p1;
    //!!p1作为const对象，不能调用普通函数，因为普通函数需要Person* const类型的this!!
	p1.func();//报错
	
    //const成员函数需要的this是const Person* const，正好可以
    p1.func2();  //正常输出
	
	return 0;
}
```

