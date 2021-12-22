---
title: static
date: 2020-07-08
tag:
  - static
  - C++
categories: "C++"
cover: https://tva1.sinaimg.cn/large/008i3skNly1gseq98n6j2j30bn0bjwfp.jpg
---

### 静态变量

* 函数中的变量，类中的变量，用static修饰

<b>函数中的静态变量</b>

当变量用static修饰时，生命周期和程序的生命周期一样。即使多次调用该函数，静态变量也只分配一次空间，而且它的值还会累计使用。这对于需要先前存储函数状态的任何应用程序有非常的作用。

```cpp
#include <iostream>
#include <string>
using namespace std;

void demo()
{
  //static var
  static int var = 0;
  cout<<var<<" ";
  
  var ++;
}

int main()
{
  for(int i = 0;i<3;i++)
  	demo();
  return 0;
}


output:
0 1 2 
```

* 每次调用demo() 都不会对var 进行初始化，它的值是累计的状态。

<b>类中的静态变量</b>

* 由于static的变量只被初始化了一次，故它在静态存储区分配了空间。因此**类中的静态变量由所有对象共享**；**不同的对象，不能有相同静态变量的多个副本。也就是这个原因，静态变量不能使用构造函数进行初始化**

```cpp
#include <iostream>
using namespace std;

class Apple
{
public:
	static int i;
  
  Apple()
  {
  		//do nothing  
  };
  
};

int main()
{
  Apple obj1;
  Apple obj2;
  obj1.i = 2;
  obj2.i = 3;
  
  cout<<obj1.i<<" "<<obj2.i<<endl;
  return 0;
}



//output:
Undefined symbols for architecture x86_64:
  "Apple::i", referenced from:
      _main in class_static-b4ba1a.o
```





* 上面尝试为多个对象创建多个静态变量的副本,最终的结果是错误的，正确的：

  类中的静态变量应由用户使用类外的类名和范围解析运算符显式初始化。



```cpp
#include <iostream>
using namespace std;

class Apple
{
 public:
  static int i;
  
  Apple(){};
  
};

int Apple::i = 1;

int main()
{
  Apple obj;
  
  cout <<obj.i<<endl;
  
  return 0;
}

//output
1
```





### 静态成员



<b>类对象为静态</b>



和变量的static属性一样，声明对象时static修饰可以让对象的生命周期直到程序结束。



```cpp
//如下程序的对象是非静态的

#include <iostream>
using namespace std;

class Apple
{
	int i ;
  public:
  	Apple()
    {
      i = 0;
      cout<<"in constructor"<<endl;
    }
  ~Apple()
  {
    cout<<"in destructor"<<endl;
  }
};

int main()
{
  
  int x = 0;
  if(x == 0)
  {
    Apple obj;
  }
  
  cout<<"end of main"<<endl;
  return 0;
}


//output:
in constructor
in destructor
end of main
  
  
//如果main函数中这样做：
  
if(x == 0)
{
  static Apple obj;
}



//output:
in constructor
end of main
in destructor
  
//main函数结束后调用析构函数，静态对象的范围贯穿程序的生命周期
```



类成员函数为静态



* 类中的静态函数和静态成员一样，也不依赖于类的对象，我们使用类名加范围解析运算符来调用静态成员函数



* 允许静态成员函数仅访问静态数据成员或者其他静态成员函数，它们无法访问类的非静态数据成员或者成员函数



```cpp
#include <iostream>
using namespace std;

class Apple
{
  public:
  static void printMSG()
  {
    cout<<"welcome"<<endl;
  }
};

int mian()
{
  Apple::printMSG();
  
  return 0;
  
}

//output
welcome
```



### 综合举例

- TestStatic.h

```cpp
#pragma once



#include <iostream>


using std::cout;
using std::endl;



class TestStatic
{
public:
	TestStatic();
	void func1();
	static void func2();
	static void func3(const TestStatic& obj);


	const static int num4 = 99;//又要实现共享，又要不可变，直接类内初始化
	int num1;
	static int num2;

	
private:
	static int num3;
};


```

- TestStatic.h

```cpp
#include "TestStatic.h"





//1.静态变量的初始化,在类内声明，在类外定义，区分作用域控制符

int TestStatic::num3 = 300;
int TestStatic::num2 = 200;



TestStatic::TestStatic()
{
	this->num1 = 100;
}



void TestStatic::func1()
{
	cout << "num1 = " << this->num1 << endl;
	cout << "通过成员函数调用num2 = " << TestStatic::num2 << endl;//类内访问private 
}

//4
void TestStatic::func2()
{
	cout << "l类中的静态函数，用于管理类中的静态数据" << endl;
}



//6
void TestStatic::func3(const TestStatic& obj)
{
	cout <<"用对象访问非静态变量"<< obj.num1 << endl;
}


```



- main.cpp

```cpp
#include "TestStatic.h"

//5
//静态成员 在编译的时候已经分配内存，那个时候对象还没有创建
//类内声明，类外定义
// className::static_func    className::static_value

//静态成员变量在各个对象中传递信息
//静态成员函数用来管理静态变量


//----------------------
//1.静态全局变量
static int dd;//静态区（全局区）.bss;=0;定义到文件结尾

namespace TT
{

	//2.静态局部变量
	static int dd2;//静态区（全局区）.bss;=0;定义到作用域结束
	//作用域结束也不会销毁，会直到程序结束
	//再次被调用的时候，值是不变的
}


//3.静态函数：限制在本文件使用，在别的文件中不可调用
//不要在CPP中声明 非static 全局函数
//不要在H中声明static函数

static void sfunc1() {

	cout << " global static func" << endl;

	cout << " TT " << TT::dd2 << endl;

}








int main(void)
{

	TestStatic ts1;
	//调用static var
	cout << "通过对象直接调用" << ts1.num2 << endl;
	cout << "通过类的作用域调用" << TestStatic::num2 << endl;
	ts1.func1();

	cout << "dd = "<<dd << endl;

	sfunc1();

	TestStatic::func2();
	TestStatic::func3(ts1);

	cout << "出了作用域访问局部static变量 dd = " << TT::dd2 << endl;

	cout << "const static value = " << ts1.num4 << endl;


	return 0;
}

```

