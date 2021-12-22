---
title: QTreeWidget的简单使用
date: 2021-06-05
tags:
  - QTreeWidget
categories:
  - Qt
cover: https://tva1.sinaimg.cn/large/008i3skNly1gtbkzmfymtj30q40hw0u8.jpg
---

### QTreeWidget

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gtbkzmfymtj30q40hw0u8.jpg" alt="image-20210810122942945" style="zoom:50%;" />

```c++
//设置水平头
    ui->treeWidget->setHeaderLabels(QStringList()<<"英雄"<<"英雄介绍");
    //加载顶层的节点
    QTreeWidgetItem *item1 = new QTreeWidgetItem(QStringList()<<"力量");
    QTreeWidgetItem *item2 = new QTreeWidgetItem(QStringList()<<"敏捷");
    QTreeWidgetItem *item3 = new QTreeWidgetItem(QStringList()<<"智力");
    //加载上去（加载新的一行）
    ui->treeWidget->addTopLevelItem(item1);
    ui->treeWidget->addTopLevelItem(item2);
    ui->treeWidget->addTopLevelItem(item3);

    //子节点
    QTreeWidgetItem *item1_1 = new QTreeWidgetItem(QStringList()<<"头节点1的子节点1"<<"test_item1_1");
    QTreeWidgetItem *item1_2 = new QTreeWidgetItem(QStringList()<<"头节点1的子节点2"<<"test_item1_2");
    QTreeWidgetItem *item1_3 = new QTreeWidgetItem(QStringList()<<"头节点1的子节点3"<<"test_item1_3");

    QTreeWidgetItem *item2_1 = new QTreeWidgetItem(QStringList()<<"头节点2的子节点1"<<"test_item2_1");
    QTreeWidgetItem *item2_2 = new QTreeWidgetItem(QStringList()<<"头节点2的子节点2"<<"test_item2_2");
    QTreeWidgetItem *item2_3 = new QTreeWidgetItem(QStringList()<<"头节点2的子节点3"<<"test_item2_3");

    QTreeWidgetItem *item3_1 = new QTreeWidgetItem(QStringList()<<"头节点3的子节点1"<<"test_item3_1");
    QTreeWidgetItem *item3_2 = new QTreeWidgetItem(QStringList()<<"头节点3的子节点2"<<"test_item3_2");
    QTreeWidgetItem *item3_3 = new QTreeWidgetItem(QStringList()<<"头节点3的子节点3"<<"test_item3_3");

    item1->addChild(item1_1);
    item1->addChild(item1_2);
    item1->addChild(item1_3);

    QList<QTreeWidgetItem*> list1;

    list1.push_back(item2_1);
    list1.push_back(item2_2);
    list1.push_back(item2_3);

    item2->addChildren(list1);

    item3->addChild(item3_1);
    item3->addChild(item3_2);
    item3->addChild(item3_3);
```



#### 测试

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gtbo1plbcsj30sw0pgdhb.jpg" alt="image-20210810141528680" style="zoom: 33%;" />

```cpp
    //设置表头
    ui->treeWidget->setHeaderLabels(QStringList()<<"Group"<<"Name"<<"Sex"<<"Score"<<
                                    "Excellent");

    //设置表头的背景颜色
    ui->treeWidget->setStyleSheet("QHeaderView::section{background:rgb(85, 181, 200);}");

    //设置表头的前景色
    QBrush brush;
    brush.setColor(QColor(255,200,0));
    brush.setStyle(Qt::SolidPattern); //Qt::SolidPattern 统一颜色
    ui->treeWidget->headerItem()->setForeground(0,brush);

    //隐藏某一列
    ui->treeWidget->hideColumn(4);

    //设置列宽
    ui->treeWidget->setColumnWidth(1,150);

    //设置表头的字体大小
    QFont font = ui->treeWidget->headerItem()->font(0);
    font.setBold(true);
    font.setPointSize(20);
    ui->treeWidget->headerItem()->setFont(0,font);

    //清除QTreeWidget的数据
    ui->treeWidget->clear();

    //添加根节点
    for(int i = 0;i<3;++i)
    {
        QTreeWidgetItem *item = new QTreeWidgetItem(QStringList()<<QString("group%1").arg(i));
        ui->treeWidget->addTopLevelItem(item);
    }

    //子节点
    QList<QTreeWidgetItem*> list;

    for(int i = 0;i<4;++i)
    {
        QTreeWidgetItem *item = new QTreeWidgetItem(QStringList()<<QString::number(i)
                                                                  <<QString("name%1").arg(i)
                                                                  <<QString("%1").arg(i%2==0?"girl":"boy")
                                                                  <<QString::number(qrand()%100+60));
        list.push_back(item);
    }

    ui->treeWidget->topLevelItem(0)->addChildren(list);

    //=============
    QTreeWidgetItem *item2 = new QTreeWidgetItem(QStringList()<<"99"<<
                                                                  "孙悟空"<<"boy"<<"9999");
    ui->treeWidget->topLevelItem(1)->addChild(item2);


    //=============
    QTreeWidgetItem *item3 = new QTreeWidgetItem();
    item3->setText(0,"1");
    item3->setText(1,"牛魔王");
    item3->setText(2,"boy");
    item3->setText(3,"100");
    ui->treeWidget->topLevelItem(2)->addChild(item3);


    //定位节点
    //第一个主节点的第三个子节点
    QTreeWidgetItem *item4 = ui->treeWidget->topLevelItem(0)->child(2);
    ui->treeWidget->scrollToItem(item4);//滚轮定位到这个节点


    //单击获取信息
    connect(ui->treeWidget,&QTreeWidget::itemClicked,this,[=](){
       //获取信息

       QTreeWidgetItem *item = ui->treeWidget->currentItem();
       int column = ui->treeWidget->currentColumn();

       //判断不是主节点，才有parent()


       if( (item != ui->treeWidget->topLevelItem(0)) &&  (item != ui->treeWidget->topLevelItem(1)) && (item != ui->treeWidget->topLevelItem(2)))
       {
           //qDebug()<<QString("%1主").arg(ui->treeWidget->indexOfTopLevelItem(item));

           qDebug()<<"主节点顺序数"<<ui->treeWidget->indexOfTopLevelItem(item->parent())+1
                    <<"子节点顺序数"<<item->parent()->indexOfChild(item)+1
                    <<"子节点列数"<<column;
       }

    });

```

