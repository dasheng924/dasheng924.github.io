---
title: QTableWidget的简单学习
date: 2021-06-01
tags:
  - QTableWidget
categories:
  - Qt
cover: https://tva1.sinaimg.cn/large/008i3skNly1gtb3biphfpj310k0q2jud.jpg
---

### QTableWidget 

-  QTableWidget是QT程序中常用的显示数据表格的空间，很类似于VC、C#中的DataGrid。说到QTableWidget，就必须讲一下它跟QTabelView的区别了。
- QTableWidget是QTableView的子类，主要的区别是QTableView可以使用自定义的数据模型来显示内容(也就是先要通过setModel来绑定数据源)，而QTableWidget则只能使用标准的数据模型，并且其单元格数据是QTableWidgetItem的对象来实现的(也就是不需要数据源，将逐个单元格内的信息填好即可)。这主要体现在QTableView类中有setModel成员函数，而到了QTableWidget类中，该成员函数变成了私有。使用QTableWidget就离不开QTableWidgetItem。QTableWidgetItem用来表示表格中的一个单元格，整个表格都需要用逐个单元格构建起来。

```cpp
 //直接构造
    //构造QTabWidget对象，设置为10行，5列
    QTableWidget *tableWidget = new QTableWidget(10,5); //10行5列


    //先有对象，再去设置对应的属性
    //下面的方法也是可以的
    QTableWidget *tableWidget2 = new QTableWidget;
    tableWidget2->setRowCount(10);
    tableWidget2->setColumnCount(5);
```

### 代码演示

![image-20210810021820128](https://tva1.sinaimg.cn/large/008i3skNly1gtb3biphfpj310k0q2jud.jpg)

#### 列相关

```cpp
ui->tableWidget->setColumnCount(5);//设置5列

    //设置表头内容
    QStringList header;
    header<<"姓名"<<"性别"<<"年龄"<<"学号"<<"平均成绩";
    ui->tableWidget->setHorizontalHeaderLabels(header);

    //设置表头不可点击,默认点击是进行排序
    ui->tableWidget->horizontalHeader()->setSectionsClickable(false);
    //设置表头是可以移动位置的
    ui->tableWidget->horizontalHeader()->setSectionsMovable(true);
    //设置表头的字体
    QFont font = ui->tableWidget->horizontalHeader()->font();
    font.setBold(true); //也就是先获取原有字体，然后加粗，然后再设置
    ui->tableWidget->horizontalHeader()->setFont(font);
    //设置列的宽度
    ui->tableWidget->horizontalHeader()->setDefaultSectionSize(300);

    //ui->tableWidget->horizontalHeader()->resizeSection(0, 150); //设置表头第一列的宽度为150
    //ui->tableWidget->horizontalHeader()->setFixedHeight(40);    //设置表头的高度
    //设置列宽策略，使列自适应宽度，所有列平均分来填充空白部分
    ui->tableWidget->horizontalHeader()->setSectionResizeMode(QHeaderView::Stretch);
    //设置列表头的样式
	ui->tableWidget>horizontalHeader()>setStyleSheet("QHeaderView::section{background:skyblue;}");
    

```

#### 行相关

```cpp
 //设置10行
    ui->tableWidget->setRowCount(10);
    //设置行的高度
    ui->tableWidget->verticalHeader()->setDefaultSectionSize(40);
    //设置垂直头不可见，不显示行号
    ui->tableWidget->verticalHeader()->setVisible(true);//是否隐藏表头

    //设置行宽策略，使列自适应宽度，所有列平均分来填充空白部分
    ui->tableWidget->verticalHeader()->setSectionResizeMode(QHeaderView::Stretch);

    //设置行表头的样式
    ui->tableWidget>verticalHeader()>setStyleSheet("QHeaderView::section{background:skyblue;}");

```



#### 整体设置

```cpp
//ui->tableWidget->setFrameShape(QFrame::NoFrame); //设置无边框
    //ui->tableWidget->setShowGrid(false); //设置不显示格子线

    //ui->tableWidget->setSelectionMode(QAbstractItemView::NoSelection);   //设置不可选择
    ui->tableWidget->setSelectionMode(QAbstractItemView::ExtendedSelection);//设置多个选中
    /*
QAbstractItemView::NoSelection 不能选择

QAbstractItemView::SingleSelection 选中单个目标

AbstractItemView::MultiSelection 选中多个目标

QAbstractItemView::ExtendedSelection QAbstractItemView::ContiguousSelection 的区别不明显，主要功能是正常情况下是单选，但按下Ctrl或Shift键后，可以多选

     */
    ui->tableWidget->setSelectionBehavior(QAbstractItemView::SelectItems); //整行选中的方式
    /*
Constant  Value  Description
QAbstractItemView::SelectItems  0  Selecting single items.选中单个单元格
QAbstractItemView::SelectRows  1  Selecting only rows.选中一行
QAbstractItemView::SelectColumns  2  Selecting only columns.选中一列
     */
    ui->tableWidget->setEditTriggers(QAbstractItemView::NoEditTriggers); //设置不可编辑
    /*
QAbstractItemView::NoEditTriggers  0  不能对表格内容进行修改
QAbstractItemView::CurrentChanged  1  任何时候都能对单元格修改
QAbstractItemView::DoubleClicked  2  双击单元格
QAbstractItemView::SelectedClicked  4  单击已选中的内容
QAbstractItemView::EditKeyPressed  8  Editing starts when the platform edit key has been pressed over an item.
QAbstractItemView::AnyKeyPressed  16  按下任意键就能修改
QAbstractItemView::AllEditTriggers  31 以上条件全包括
    */
```

#### 单元格设置

```cpp
 //单元格设置字体颜色和背景
    QTableWidgetItem *item = new QTableWidgetItem("Apple");
    item->setBackgroundColor(QColor(0,60,10));
    item->setTextColor(QColor(100,120,100));
    item->setFont(QFont("Helvetica"));
    ui->tableWidget->setItem(0,0,item);

    //设置单元格对齐方式
    item->setTextAlignment(Qt::AlignHCenter|Qt::AlignVCenter);
    /*
Constant Value Description
Qt.AlignLeft 0x0001 Aligns with the left edge.
Qt.AlignRight 0x0002 Aligns with the right edge.
Qt.AlignHCenter 0x0004 Centers horizontally in the available space.
Qt.AlignJustify 0x0008 Justifies the text in the available space.

Constant Value Description
Qt.AlignTop 0x0020 Aligns with the top.
Qt.AlignBottom 0x0040 Aligns with the bottom.
Qt.AlignVCenter 0x0080 Centers vertically in the available space.

    */





    //所有的单元格设置字体
    ui->tableWidget->setFont(QFont("Helvetica"));

    //单元格的合并
    //ui->tableWidget->setSpan(0,0,3,1);

    //设置单元格的大小
    //ui->tableWidget->setColumnWidth(1,50); //第二列设置为50的高度
   // ui->tableWidget->setRowHeight(0,50);//第1行设置为50的高度

    ui->tableWidget->resizeColumnsToContents();//列的大小与内容相匹配
    ui->tableWidget->resizeRowsToContents();




    //获取单元格的内容,信号  itemClicked (QTableWidgetItem *)
    connect(ui->tableWidget,&QTableWidget::itemClicked,this,[=](QTableWidgetItem *item){
        qDebug()<<item->text();

    });


    //动态的对数据进行插入
    for(int i = 0;i<10;++i)
    {
        int rowNum = ui->tableWidget->rowCount();
        ui->tableWidget->insertRow(rowNum);

        //新建item
        QTableWidgetItem *item1 = new QTableWidgetItem;
        QTableWidgetItem *item2 = new QTableWidgetItem;
        QTableWidgetItem *item3 = new QTableWidgetItem;
        QTableWidgetItem *item4 = new QTableWidgetItem;
        QTableWidgetItem *item5 = new QTableWidgetItem;

        //设置字体显示风格：居中
        item1->setTextAlignment(Qt::AlignHCenter |  Qt::AlignVCenter);
        item2->setTextAlignment(Qt::AlignHCenter |  Qt::AlignVCenter);
        item3->setTextAlignment(Qt::AlignHCenter |  Qt::AlignVCenter);
        item4->setTextAlignment(Qt::AlignHCenter |  Qt::AlignVCenter);
        item5->setTextAlignment(Qt::AlignHCenter |  Qt::AlignVCenter);


        //header<<"姓名"<<"性别"<<"年龄"<<"学号"<<"平均成绩";
        item1->setText(QString("name%1").arg(i));
        item2->setText(QString("sex%1").arg(i));
        item3->setText(QString("age%1").arg(i));
        item4->setText(QString("id%1").arg(i));
        item5->setText(QString("score%1").arg(i));



        //设置item
        ui->tableWidget->setItem(rowNum, 0, item1);
        ui->tableWidget->setItem(rowNum, 1, item2);
        ui->tableWidget->setItem(rowNum, 2, item3);
        ui->tableWidget->setItem(rowNum, 3, item4);
        ui->tableWidget->setItem(rowNum, 4, item5);


    }

    //循环清空行
    for(int i = 0; i < 10; ++i)
    {
        //参数为0，不是i，自动delete里面的item
        ui->tableWidget->removeRow(0);
    }


	    //单元格里面可以加入控件
//    QComboBox *comboBox = new QComboBox();
//    comboBox->addItem("Y");
//    comboBox->addItem("N");
//    ui->tableWidget->setCellWidget(0,0,comboBox);

```

