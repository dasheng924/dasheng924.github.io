---
title: Qt中Json的使用
date: 2021-4-28
tags:
  - JSON
categories:
  - Qt
cover: https://tva1.sinaimg.cn/large/008i3skNly1gss5e8m7vmj305u030a9w.jpg
---

### JSON 

- 一种数据格式，与语言无关
- 层次结构清晰，方便阅读和读取识别

### JSON分类

#### JSON数组

- `[]`里面放元素，元素的类型包括 {% label 整形 green %},{% label 浮点形 green %},{% label 字符串 green %},{% label 布尔 green %},{% label JSON数组 green %},{% label  JSON对象 green %}，{% label 空值Null green %}.

```json
//整形
[1,3,4,56]
//字符串
["aaa","bbb","ccc","ddd"]
//混合
[12,12.22,true,false,"hello",null]
```

```json
//json 数组的嵌套
[
  ["cat","dog","pig"]
  ["apple","good",12]
  [true,null]
]

```

#### JSON对象

- `{}`来描述JSON对象，每一个JSON对象可以存粗多个元素，每个元素都是键值对
- 键必须是字符串类型，同一层级的键不可重复
- value类型 {% label 整形 green %},{% label 浮点形 green %},{% label 字符串 green %},{% label 布尔 green %},{% label JSON数组 green %},{% label  JSON对象 green %}，{% label 空值Null green %}.

```json
{
  "name":"Alex",
  "Sex":"man",
  "Age":22,
  "Family":{
    	"Father":"AAA",
    	"Month":"BBB"
  }
  "IsAlive":false
}
```

### {% label 在一个JSON文件中只有一个JSON数组或者JSON对象的根节点，不允许同时存储多个并列根节点 pink %}

### Qt 中的JSON类

| JSON类                         | 详细                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| {%label QJsonDocument green %} | JSON文档类，支持以UTF-8编码的文本或者Qt 的二进制格式读取和写入 |
| {%label QJsonArray green %}    | JSON数组是一个值列表，支持插入和删除QJsonValue 来操作列表    |
| {%label QJsonObject green %}   | JSON对象是一个键值对列表，支持插入和删除QJsonValue 来操作列表 |
| {%label QJsonValue green %}    | 封装了JSON支持的数据类型                                     |

#### QJsonValue

- 可以封装基础数据类型
- 布尔类型 `QJsonValue::Bool`
- 浮点类型 `QJsonValue::Double`
- 字符串类型 `QJsonValue::String`
- JSON 数组 `QJsonValue::Array`
- JSON 对象 `QJsonValue::Object`
-  空值类型 `QJsonValue::Null`

```c++
//1.构造函数
QJsonValue 的构造函数封装了基本的数据类型

//2.提供了许多判断数据类型的函数
isArray()
isObject()
isBool()
  
//3.提供了转换为基础数据类型的函数
toArray()
toBool()
toDouble()
```

#### QJsonObject

- 封装了Json中的对象，可以存储多个键值对，键值都为字符串类型，值为`QJsonValue`

- 创建空的Json对象

```c++
QJsonObject::QJsonObject(); //构造空对象
```

- 将键值对添加到空对象中

```c++
iterator QJsonObject::insert(const QString &key,const QJsonValue &value);
```

- 获取键值对的个数

```c++
int QJsonObject::count() const;
int QJsonObject::size() const;
int QJsonObject::length() const;
```

- 通过key得倒value

```c++
QJsonValue QJsonObject::value(const QString &key) const ;
QJsonValue QJsonObject::operator[](const QString &key) const;
```

- 删除键值对 

```c++
void QJsonObject::remove(const QString &key);
QJsonValue QJsonObject::take(const QString &key); //删除且返回删除的value值
```

- 通过key查找

```c++
iterator QJsonObject::find(const QString &key);
bool QJsonObject::contains(const QString &key) const ;
```

- 获取所有的键列表

```c++
QStringList QJsonObject::keys() const;
```



#### QJsonArray

- QJsonArray 封装了Json中的数组，可以存储多个元素，所有的元素类型为`QJsonvalue`。

- 创建JSON数组

```c++
QJsonArray::QJsonArray()
```

- 添加数据

```c++
append //尾部追加
insert //插入到i的位置之前
prepend //添加到数组头部
push_back //添加到尾部
push_front//添加到头部
 
```



- 数组元素个数

```c++
int QJsonArray::count() const ;
int QJsonArray::size() const ;
```

- 取值

```c++
QJsonValue QJsonArray::at(int i) const;
QJsonValue QJsonArray::first() const;
QJsonValue QJsonArray::last() const;
QJsonValueRef QJsonArray::operator[](int i);
```

- 删除

```c++
iterator QJsonArray::erase(iterator it);    // 基于迭代器删除
void QJsonArray::pop_back();           // 删除尾部
void QJsonArray::pop_front();          // 删除头部
void QJsonArray::removeAt(int i);      // 删除i位置的元素
void QJsonArray::removeFirst();        // 删除头部
void QJsonArray::removeLast();         // 删除尾部
QJsonValue QJsonArray::takeAt(int i);  // 删除i位置的原始, 并返回删除的元素的值
```



#### QJsonDocument

- 封装了一个完整的JSON文档，支持UTF-8编码的文本和Qt自己的二进制个数的数据读取和写入
- `QJsonObject`和`QJsonArray`中的数据是不能直接转化为字符串的，需要先转化为JSON文档。

```json
{
    "server": {
        "ip": "127.0.0.1",
        "name": "Alex",
        "port": 9999
    }
}
```



#####  `QJsonObject`和`QJsonArray` ------》字符串

```c++
//1.创建QJsonDocument 对象
QJsonDocument::QJsonDocument(const QJsonObject &object);
QJsonDocument::QJsonDocument(const QJsonArray &array);
//2.得倒二进制格式数据或者json字符串
QByteArray QJsonDocument::toBinaryData() const ;
QByteArray QJsonDocument::toJson(JsonFormat format = Indented) const;
//3.得倒数据进行数据传输后者数据持久化
//---------------------------------------------
//创建JSON对象
    QJsonObject obj;
    //创建JSON子对象
    QJsonObject sub;
    sub.insert("ip",QJsonValue("127.0.0.1"));
    sub.insert("port",QJsonValue(9999));
    obj.insert("server",QJsonValue(sub));

    //内存中的数据写到文件
    QJsonDocument doc(obj);//得倒document对象
    //将json对象转化为字符串
    QByteArray data = doc.toJson();
    //把字符串写入文件
    QFile file("/Users/sunguosheng/Downloads/temp.json");
    file.open(QIODevice::WriteOnly);
    file.write(data);
    file.close();
```



##### 字符串-----》`QJsonObject`和`QJsonArray` 

```c++
//1.JSON 格式字符串转化为QJsonDocument类对象
[static] QJsonDocument QJsonDocument::fromBinaryData(const QByteArray &data, DataValidation validation = Validate);
// 参数文件格式的json字符串
[static] QJsonDocument QJsonDocument::fromJson(const QByteArray &json, QJsonParseError *error = Q_NULLPTR);

//2.JSON文档转化为 JSON数组/对象
// 判断文档对象中存储的数据是不是数组
bool QJsonDocument::isArray() const;
// 判断文档对象中存储的数据是不是json对象
bool QJsonDocument::isObject() const
    
// 文档对象中的数据转换为json对象
QJsonObject QJsonDocument::object() const;
// 文档对象中的数据转换为json数组
QJsonArray QJsonDocument::array() const;
//------------------------------------------
QFile file("/Users/sunguosheng/Downloads/temp.json");
    file.open(QIODevice::ReadOnly);
    QByteArray data = file.readAll();

    QJsonDocument doc = QJsonDocument::fromJson(data);
    if(doc.isObject())
    {
        QJsonObject obj = doc.object();
        QJsonValue value = obj.value("server");
        if(value.isObject())
        {
            QJsonObject sub = value.toObject();
            QString ip = sub.value("ip").toString();
            int port = sub.value("port").toInt();
            qDebug()<<ip<<port;
        }

    }
```



### 参考链接: https://subingwen.cn/qt/qt-json/