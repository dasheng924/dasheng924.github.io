---
title: decltype
date: 2021-06-18
Tags:
  - declType
categories:
  - C++
cover: https://tva1.sinaimg.cn/large/008i3skNly1gtl4v038nbj60cx07egli02.jpg
---

### decltype

- 和auto不同的是，推导的变量不需要初始化
- decltype的推导是在编译期完成的，表达式不会计算，只会取其类型

### 推导规则

- 表达式为普通变量或者普通表达式或者类表达式，在这种情况下，使用 decltype 推导出的类型和表达式的类型是一致的

```c++
#include <iostream>
#include <string>
using namespace std;

class Test{
public:
	string text;
  static const int value = 99;//static静态成员变量
};

int main()
{
  int num = 99;
  decltype(num) a = num;//a-->int
  
  const int &y = num;
  decltype(y) b = num;//b-->const int &
  
  decltype(Test::value) c = 0;//c-->const int
  
  
  return 0;
}
```

- 表达式是函数调用，使用decltype推导出的类型和函数的返回值保持一致
  - 返回的右值，不属于某个类，const ,volatile需要丢掉

```c++
class Test{...};
//函数声明
int func_int();                 // 返回值为 int
int& func_int_r();              // 返回值为 int&
int&& func_int_rr();            // 返回值为 int&&

const int func_cint();          // 返回值为 const int
const int& func_cint_r();       // 返回值为 const int&
const int&& func_cint_rr();     // 返回值为 const int&&

const Test func_ctest();        // 返回值为 const Test

//decltype类型推导
int n = 100;
decltype(func_int()) a = 0;		//a-->int
decltype(func_int_r()) b = n;	//b-->int&
decltype(func_int_rr()) c = 0;	//c-->int &&
decltype(func_cint())  d = 0;	//d-->int
decltype(func_cint_r())  e = n;	//e-->const int&
decltype(func_cint_rr()) f = 0;	//f-->const int&&
decltype(func_ctest()) g = Test();//g-->const Test

```

- 表达式是一个左值，或者被括号 ( ) 包围，使用 decltype 推导出的是表达式类型的引用（如果有 const、volatile 限定符不能忽略）。

```c++
#include <iostream>
#include <vector>
using namespace std;

class Test
{
public:
    int num;
};

int main() {
    const Test obj;
    //带有括号的表达式
    decltype((obj.num)) b = a;//const int &
    //加法表达式
    int n = 0, m = 0;
    decltype(n = n + m) d = n;//int&
    return 0;
}
```

### 应用

- 模板

```c++
#include <iostream>
#include <list>
using namespace std;

//对容器提供遍历
template<typename T>
class Container{
public:
    void show(T &t)
    {
        for(m_it = t.begin();m_it != t.end();++m_it)
        {
            cout<<*m_it<<endl;
        }
    }

private:
    typename T::iterator m_it1;
    decltype(T().begin()) m_it2;
};

int main()
{
    list<int> l{1,32,3,1,56,7};
    Container<list<int>> c;
    c.show(l);


    return 0;
}
```

- 返回值类型后置

```c++
#include <iostream>

using namespace std;

//返回值直接写auto是可以的
//返回值后置的情况下，auto推导的是后面的decltype(v+u)
template<typename V,typename U>
auto add(V v,U u) -> decltype(v+u)
{
    return v+u;
}

int main()
{
    int x = 100;
    double y = 3.1415;
    auto ret = add<int,double>(x,y);
    cout<<ret<<endl;
}
```

