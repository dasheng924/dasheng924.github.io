---
title: auto
date: 2021-06-20
Tags:
  - auto
categories:
  - C++
cover: https://tva1.sinaimg.cn/large/008i3skNly1gtkcpy4yohj60go0c7dgf02.jpg
---

### auto

- 不是新的数据类型，是类型声明的占位符
- auto声明的变量必须要初始化

```c++
auto 变量名 = 变量值;
```

### 推导规则

```c++
int temp = 110;
auto *a = &temp; //a-->int* ;auto-->int	
auto b = &temp;		//b-->int*;auto->int*
auto &c = temp;		//c-->int&;auto-->int
auto d = temp;		//d-->int ;auto-->int
```

```c++
//当变量是指针或者引用类型时，推导的结果中会保留 const、volatile 关键字
int tmp = 250;
const auto a1 = tmp;//a1-->const int ;auto-->int
auto a2 = a1;//a2按道理来说为const int，但是a2不是引用也不是指针，故去掉const属性,a2-->int;auto-->int
const auto &a3 = tmp;//a3-->const int&,auto-->int,const本来就有的，所以不需要忽略，和a2对比
auto &a4 = a3;//a4-->const int &;auto-->const int
```

### 限制

- 不可做函数的参数，因为函数形参在调用时才会传递实参数，而auto 需要赋值才会推导

```c++
void func(auto a) //error
{
  
}
```

- 不能用于类的非静态成员(可以用于类的静态成员)

```c++
class Test{
  auto a1 = 100;//error
  static auto a2 = 100;//error,静态成员必须类外初始化
  static const auto a3 = 100;//ok
};
```

-  不能使用auto定义数组

```c++
int func()
{
    int array[] = {1,2,3,4,5};  // 定义数组
    auto t1 = array;            // ok, t1被推导为 int* 类型
    auto t2[] = array;          // error, auto无法定义数组
    auto t3[] = {1,2,3,4,5};;   // error, auto无法定义数组
}
```

- 无法使用auto推导出模版参数

```c++
template <typename T>
struct Test{};

int func()
{
  Test<double> t; 
  Test<auto> t1 = t; //error
  return 0;
}
```

### 应用

#### STL容器遍历

```c++
map<int,string> p;
//loop
for(auto it = p.begin();it != p.end();++it)
{
  
}
```

```c++
map<int,string> p;
//loop
for(auto it:p)
{
  
}
```

#### 泛型编程

```c++
#include <iostream>
#include <string>

using namespace std;

class T1{
public:
    static int get()
    {
        return 10;
    }
};

class T2{
public:
    static string  get(){
        return "hello man";
    }
};

template<class T>
void func(void)
{
    auto v = T::get();
    cout<<"v = "<<v<<endl;
}
//不使用auto:
template<class V1,typename V2>
void func2(void)
{
  V2 v = V1::get();
  cout<<"v = "<<v<<endl;
}
  
int main()
{
    func<T1>();
    func<T2>();
  
  	func2<T1,int>();
  	func2<T2,string>();

    return 0;
}

//out
/*
v = 10
v = hello man
*/
```