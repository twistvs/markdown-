#### 1.构造与赋值

```c++
# include <iostream>
# include <map>
using namespace std;


void printmap(map<int, int>& m)
{
	//it是迭代器
	for (auto it = m.begin(); it != m.end(); it++)
	{
		cout << it->first << " : " << it->second << endl;
	}
}

void test1()
{
	//默认构造函数
	map<int, int> map1;
	map1.insert({ 1, 1 });        //map中的元素是pair类型的
	printmap(map1);

	//拷贝构造函数
	map map2(map1);
	printmap(map2);

	//赋值操作
	map <int, int> map3;
	map3 = map2;
	printmap(map3);
}

int main()
{
	test1();

	return 0;
}
```

- map中的所有元素都是==pair==类型，pair的first称为==键值==，pair的second称为==实值==
- map中的元素会按照键值从小到大的顺序进行排序

------

#### 2.大小与交换

```c++
# include <iostream>
# include <map>
using namespace std;

//大小
void test1()
{
	map<int, int> m1;
	m1.insert({ 1, 1 });
	if (m1.empty())
	{
		cout << "m1是空的" << endl;
	}
	else
	{
		cout << "m1不是空的" << endl;
		cout << "m1的大小为" << m1.size() << endl;
	}
 }

//交换
void test2()
{
	map <int, int> m1;
	m1.insert({ 1, 1 });

	map <int, int> m2;
	m2.insert({ 2, 2 });

	m1.swap(m2);
	cout << m1.begin()->first << endl;
	cout << m2.begin()->first << endl;
}

int main()
{
	test1();
	test2();
	return 0;
}
```

- `empty`，`size`，`swap`对map依旧适用

#### 3.插入与删除

```c++
# include <iostream>
# include <map>
using namespace std;

void printmap(map<int, int>& m)
{
	//it是迭代器
	for (auto it = m.begin(); it != m.end(); it++)
	{
		cout << it->first << " : " << it->second << endl;
	}
}

//map的插入
void test1()
{
	map<int, int> m1;
	//1.直接用pair类型插入
	m1.insert(pair<int, int>(1, 1));     //写出pair的完整类型
	m1.insert(pair(2, 2));               //不写pair的类型，根据(2, 2)自动推导类型
	m1.insert({ 3, 3 });                 //不写pair，花括号自动识别为pair

	//2.用make_pair函数插入
	map<int, int> m2;
	m2.insert(make_pair(4, 4));       //std::makepair返回一个pair

	//3.用map的value_type来代替pair<int, int>
	map<int, int> m3;
	m3.insert(map<int, int>::value_type(5, 5));

	//4.用key插入
	map<int, int>m4;
	m4[6] = 6;                     //不推荐用来插入，key索引最好用来访问值
	cout << m4[6] << endl;

	printmap(m1);
	printmap(m2);
	printmap(m3);
	printmap(m4);
}

//map的删除
void test2()
{
	map<int, int> m1;
	m1.insert({ 1, 1 });
	m1.insert({ 2, 2 });
	m1.insert({ 3, 3 });
	m1.insert({ 4, 4 });
	m1.insert({ 5, 5 });

	//1.用迭代器删除
	m1.erase(m1.begin());

	//2.用key删除
	m1.erase(3);
	printmap(m1);

	//3.删除一个范围
	m1.erase(m1.begin(), m1.end());
	printmap(m1);

	//4.清空
	m1.clear();
}

int main()
{

	/*test1();*/
	test2();
	return 0;
}
```

- `insert`  插入
- `erase`   删除
- `clear`   清空

#### 4.查找和统计

```c++
# include <iostream>
# include <map>
using namespace std;

void printmap(map<int, int>& m)
{
	//it是迭代器
	for (auto it = m.begin(); it != m.end(); it++)
	{
		cout << it->first << " : " << it->second << endl;
	}
}

void test1()
{
	map<int, int> m1;
	m1.insert({ 1, 1 });
	m1.insert({ 2, 2 });
	m1.insert({ 3, 3 });
	m1.insert({ 4, 4 });
	
	//find返回值是迭代器
	map<int, int>::iterator pos = m1.find(3);
	m1.erase(pos);
	printmap(m1);
	map<int, int>::iterator pos2 = m1.find(20);
	if (pos2 == m1.end())                          //如果找不到指定的元素，会返回end迭代器，end是最后一个元素的下一个位置
	{
		cout << "未找到key为20的元素" << endl;
	}

	//count统计key的个数
	int num = m1.count(2);
	cout << num << endl;            //由于map的key不允许重复，所以要么是0，要么是1；multimap可以统计出多个key
}

int main()
{

	test1();

	return 0;
}
```

- `find`   查找指定的key，如果找到了，返回迭代器；如果没找到，返回end()
- `count`    统计指定的key的数量，map只有可能是0或者1，因为map不允许key出现重复；multimap允许key重复，所以可能大于1