---
title: final和override
date: 2021-06-18
Tags:	
  - final
categories:
  - C++
cover: https://tva1.sinaimg.cn/large/008i3skNly1gtl95xnobyj608c06iglh02.jpg
---

### final

- 限制某个类不能继承，或者某个虚函数不能被重写
- final放到类或者函数的后面

### 修饰函数

- 只能修饰虚函数，这样能阻止子类重写父类的函数

```c++
#include <iostream>
using namespace std;

class Base{
public:
    virtual void test() final
    {
        cout<<"Base test()"<<endl;
    }
    virtual void func()
    {
        cout<<"base func()"<<endl;
    }
};

class Child1: public Base{
public:
    //void test(){} //不允许被重写
    virtual  void func() override
    {

    }
};
```

```c++
class Base final {
public:
    virtual void test(){
        cout<<"Base test()"<<endl;
    }
};

//Child 不可继承Base
class Child::public Base{
    
};

```



