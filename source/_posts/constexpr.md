---
title: constexpr
date: 2021-06-18
Tags:
  - constexpr
categories:
  - C++
cover: 
---

### 只读的变量和常量

```c++
//只读和常量 这两个概念是不一样的

int func(const int num)//num是一个只读变量
{
    const int count = 22;//count 是一个常量

    //定义一个数组
    //int array[num];//error,静态数组不能以变量做数组大小
    int array[count];//ok

    //引用的例子
    int a1 = 100;
    int a2 = 200;

    const int &b = a1; //b 作为a1 的别名,不可以通过b修改a1
    //b = a2;//error

    a1 = 1314;
    cout<<b<<endl; //b = 1314,可以通过a的修改影响到b
}
```

### constexpr修饰变量

- `const` `constexpr`都可以用来修饰常量；只不过建议const 修饰只读的属性，constexpr 修饰常量

```c++
//修饰基本数据类型
const int i = 100;// 是一个常量表达式
const int j = i+100;// 是一个常量表达式

constexpr int i = 99;// 是一个常量表达式
constexpr int j = i+1;// 是一个常量表达式
```

- 修饰自定义类型

```c++
constexpr struct People{
  int id;
  string name;
};

//上面的例子是错误的，不能在定义的时候，直接用constexpr修饰结构体

//正确的写法：
struct People{
  int id;
  string name;
};

constexpr People p{1,"apple"};//用constexpr修饰结构体定义的变量，最终这个表达式就是一个常量表达式，因为结构体的成员值都是确定的

constexpr int id = p.id;
constexpr string name = p.name;

p.id++;//error，常量不可修改

```

### constexpr 修饰函数

- 修饰各种类型的函数，只要满足要求，就是一个常量表达式函数

#### 修饰普通函数

- 当有返回值时，需要`return`的是常量

```c++
constexpr void func1(){int a = 100; };  //ok

constexpr int func2(int num){int a = 100;return (num+a);}//丢失了constexpr的属性

```

```c++
constexpr int add(int x, int y)
{
    return x + y;
}
int a = 10;
int b = 20;
int c = add(a, b);    //此调用无法利用constexpr在编译时期优化
```

- 函数使用前，必须在其之前的位置有过定义

```c++
#include <iostream>
using namespace std;

constexpr int func1(); //需要把定义放到函数使用之前
int main()
{
    constexpr int num = func1();	// error
    return 0;
}

constexpr int func1()
{
    constexpr int a = 100;
    return a;
}
```

- 函数体不能太复杂，不能有逻辑运算（using,typedef,static_assert,return除外）

#### 修饰模板函数

- 函数模板：是一个模板，还没有具体化
- 模板函数：通过函数模板具体化了类型

```c++
#include <iostream>
using namespace std;

struct People{
    const char * name;
    int age;
};

//定义一个函数模板
template<typename T>
constexpr T show(T t){
    return t;
}

int main()
{
    People p{"apple",12};

    //普通函数
    People ret = show(p);//p是变量，实例化后的函数不是一个constexpr表达式函数
    cout<<"name = "<<ret.name<<"\tage = "<<ret.age<<endl;

    //表达式常量函数
    constexpr int ret1 = show(250);  //常量表达式
    cout<<"ret1 = "<<ret1<<endl;

    //常量结构体实例
    constexpr People p2{"good",23};//常量表达式
    constexpr People p3 = show(p2);
    cout<<"p3.name = "<<p3.name<<"\tp3.age = "<<p3.age<<endl;

    return 0;
}
```

#### 修饰构造函数

- 构造函数体为空
- 采用初始化列表为各个成员赋值

```c++
#include <iostream>
using namespace std;

struct Person {
    constexpr Person(const char* p, int age) :name(p), age(age)
    {
    }
    const char* name;
    int age;
};

int main()
{
    constexpr struct Person p1("luffy", 19);
    cout << "luffy's name: " << p1.name << ", age: " << p1.age << endl;
    return 0;
}
```





