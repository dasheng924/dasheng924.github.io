---
title: Qt中的http使用
date: 2021-06-20
Tags:
  - http
categories:
  - Qt
cover: https://tva1.sinaimg.cn/large/008i3skNly1gtc9vru8gcj60h60b4glv02.jpg
---

### 注意

- 项目的`pro`文件里面需要加上

```c++
QT       += core gui network
```



###  `QNetworkAccessManger`

- 程序可以使用这个类的对象在网络上发送请求
- 请求我们设置为`GET` 和 `POST`
- 在一个程序里面，这个对象，最好只有一个，可以设置成类的私有成员变量

```c++
//xxx.h
private:
	//QNetworkAccessManager类允许应用程序发送网络请求和接收网络应答。
    QNetworkAccessManager *manager;

//main.cpp
 manager = new QNetworkAccessManager(this); 
```



- `QNetworkAccessManager`的构造函数

```c++
QNetworkAccessManager(QObject *parent = Q_NULLPTR)
```

- 几种请求方法

```c++
QNetworkReply *get(const QNetworkRequest &request);

QNetworkReply *post(const QNetworkRequest &request, QIODevice *data);
QNetworkReply *post(const QNetworkRequest &request, const QByteArray&data);
```

- 需要关注的信号

```c++
void finished(QNetworkReply *reply);

```

--------

### `QNetworkRequest`

- 构造函数

```c++
QNetworkRequest(const QUrl &url = QUrl()) ;
```

- 需要关注的成员函数

```c++
//设置请求头，Qt预设了几种类型

void setHeader(KnownHeaders header, const QVariant &value);

//对应于 HTTP Content-Type 标头并包含一个包含媒体 (MIME) 类型和任何辅助数据（例如，字符集）的字符串。
//QNetworkRequest::ContentTypeHeader

//---------
//对应于 HTTP Content-Length 标头，包含传输数据的字节长度。
//QNetworkRequest::ContentLengthHeader
//----------
//HTTP 客户端发送的 User-Agent 标头。
//QNetworkRequest::UserAgentHeader

//----------------------------------------
//如果没有预设的请求头，可以自定义
void QNetworkRequest::setRawHeader(const QByteArray &headerName, const QByteArray &headerValue)；
  
//举例
request.setRawHeader(QByteArray("Last-Modified"), QByteArray("Sun, 06 Nov 1994 08:49:37 GMT"));

//----------------------------------------
//设置请求的地址
void QNetworkRequest::setUrl(const QUrl &url)；

```

---



### `QNetworkReply`

- 主要用来接受服务器的返回数据

- 重要的成员函数

```c++
//获取服务器错误
NetworkError error() const ; 
```

- 重要的信号

```c++
//下行的数据 
void downloadProgress(qint64 bytesReceived, qint64 bytesTotal);
//上行的数据
void uploadProgress(qint64 bytesSent, qint64 bytesTotal);
//数据传输完成 
void finished();

```



### `GET`列子

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gtc9vru8gcj60h60b4glv02.jpg" alt="image-20210811025059531" style="zoom:50%;" />

```c++
//构造函数里面
    m_manager = new QNetworkAccessManager(this);
    connect(ui->pushButton,&QPushButton::clicked,this,[=](){
        QString str = ui->lineEdit->text();
        startRequest(QUrl(str));
    });

//槽函数

void Widget::startRequest(const QUrl &requestUrl)
{
    const QUrl url = requestUrl;
    QNetworkRequest request;
    request.setUrl(url);
    request.setAttribute(QNetworkRequest::FollowRedirectsAttribute,true);
    request.setRawHeader("Usr-Agent","macOS/11.5.1 (20G80) CalendarAgent/954");
    request.setRawHeader("Accept","*/*");
    request.setHeader(QNetworkRequest::ContentTypeHeader,"application/json");

    QNetworkReply *reply = m_manager->get(request);

    connect(reply,&QNetworkReply::finished,this,[=]()mutable{
        if(reply->error() == QNetworkReply::NoError)
        {
            //状态码
            int status = reply->attribute(QNetworkRequest::HttpStatusCodeAttribute).toInt();
            ui->textEdit->setPlainText(QString::number(status));
            //reason
            QString reason = reply->attribute(QNetworkRequest::HttpReasonPhraseAttribute).toString();
            ui->textEdit->append(reason);
            reply->deleteLater();
            reply = nullptr;
        }
        else{

            reply->deleteLater();
            reply = nullptr;

        }
    });

}
```



####  `POST`

```c++
QNetworkReply *reply = m_manager->post(request,data);

//request 是QNetworkRequest 对象，需要设置好请求头的信息和URL
//服务器和客户端之间的数据传输格式可以商定

//JSON  request.setHeader(QNetworkRequest::ContentTypeHeader,"application/json");


```



