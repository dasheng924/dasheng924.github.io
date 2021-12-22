---
title: Qt创建项目的几种方式（由渐入深）
date: 2021-10-17
tags:
  - Qt project
categories:
  - Qt
cover:	https://tva1.sinaimg.cn/large/008i3skNly1gvildx00erj608u0a0dfu02.jpg
---

###  直接一个`main.cpp`搞定

- 创建一个空的`qmake`项目，会生成一个对应的`.pro`文件     

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gvigcai7etj61c00u077f02.jpg" alt="image-20211017174828736" style="zoom: 33%;" />

- 修改`.pro`文件，添加

`greaterThan(QT_MAJOR_VERSION,4):QT += widgets`

- 添加新的文件`main.cpp`

```c++
#include <QApplication>
#include <QDialog>
#include <QObject>
#include <QLabel>

int main(int argc,char *argv[])
{
  QApplication a(argc,argv);
  QDialog w;
  QLabel label(&w);
  label.setText(QObject::tr("hello world"));
  w.show();
  
  return a.exec();
}
```

- Qt Create 直接编译运行就可以出现桌面窗口。

- 或者在终端编译运行

  ```bash
  1.创建出空的项目目录
  2.把上面的`main.cpp`加入到目录中，`qmake -project` 生成对应的pro文件
  3.修改Pro文件，加上对应的`greaterThan(QT_MAJOR_VERSION,4):QT += widgets`
  4.`qmake`生成对应的Makefile文件
  5.在Mac下面，会生成对应的app文件，`open xxx.app`打开对应的软件
  ```

  ![image-20211017194540017](https://tva1.sinaimg.cn/large/008i3skNly1gvijq6tyaqj61am0aun0u02.jpg)

### 借助`ui`文件实现界面的搭建

- 在原有的项目上添加新的文件

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gvijvpsxuzj61c90u0diw02.jpg" alt="image-20211017195058076" style="zoom:33%;" />

- 构建出对应的界面，也就是dialog上面加一个label，两者是网格布局.保存UI，进行构建，会在对应的debug目录生成`ui_dialog.h`文件

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gvijwyu0jkj60mq0gsaaw02.jpg" alt="image-20211017195211182" style="zoom:33%;" />

- 修改原有的`main.cpp`文件

```cpp
#include "ui_dialog.h"

int main(int argc,char *argv[])
{
    QApplication a(argc,argv);//建立唯一的应用程序对象
    QDialog w;

    Ui::HelloDialog ui;
    ui.setupUi(&w);//生成界面,选择模板是Dialog,所以就以dialog做参数
    w.show();
  
    return a.exec();
}
```

- 编译运行，生成可执行程序

- 或者在命令行进行文件的编译和运行

```bash
#在Mac环境下

1.新建项目目录
2.拷贝修改过的`main.cpp` `ui_dialog.ui`文件
3.`uic -o ui_dialog.h ui_dialog.ui` #根据UI文件生成对应的头文件
3.`qmke -project`
4.`qmake`
5.`make`
```



### 自定义C++类

- 新建一个空的qmake工程
- 修改pro文件`greaterThan(QT_MAJOR_VERSION,4):QT += widgets`
- 新建一个类，继承`QDialog`
- 新建`main.cpp`
- 添加上面的ui 文件
- 修稿类文件,`main.cpp`文件
- 编译运行，生成可执行程序

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gvika1n7kjj61ca0u0acb02.jpg" alt="image-20211017200445051" style="zoom:33%;" />

```c++
//main.cpp
#include <QApplication>
#include "hellodialog.h"

int main(int argc,char *argv[])
{
    QApplication a(argc,argv);
    
    HelloDialog h;
    h.show();
    
    
    return a.exec();
    
}
```

```c++
//hellodialog.h

#ifndef HELLODIALOG_H
#define HELLODIALOG_H

#include <QDialog>

//前置声明,加快编译,对应的头文件可以放到.cpp文件,因为次=此文件中只用到对应的对象指针
namespace Ui {
class HelloDialog;
}

class HelloDialog : public QDialog
{
    Q_OBJECT
public:
    explicit HelloDialog(QWidget *parent = 0);
    ~HelloDialog();

private:
    Ui::HelloDialog *ui;
};

#endif // HELLODIALOG_H

```

```c++
//hellodialog.cpp
#include "hellodialog.h"
#include "ui_dialog.h"  //UI文件对应的.h文件

HelloDialog::HelloDialog(QWidget *parent):QDialog(parent),ui(new Ui::HelloDialog) //继承要初始化对应的父对象 ,同时初始化了自己的成员变量
{
    ui->setupUi(this /*new HelloDialog*/);//表示为当前类所代表的对话框创建界面
}


HelloDialog::~HelloDialog()
{

}
```



### 设计师界面类

- 新建`qmake empty project`
- 添加设计师界面类

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gvil7v3u6hj61cc0u0q6002.jpg" alt="image-20211017203714786" style="zoom:33%;" />

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gvil877g0pj616p0u076h02.jpg" alt="image-20211017203734150" style="zoom:33%;" />

- 生成对应的文件如下，和自己用C++类是一样的效果，也就是说Qt Create为我们做了许多事情

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gvil8rvzwsj616g0u0dhy02.jpg" alt="image-20211017203807368" style="zoom:33%;" />