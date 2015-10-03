# Qcom

1.以前必须先使用open()函数打开串口，再进行配置；而现在打开串口和配置串口没有顺序要求；

2.以前在Linux下面必须使用查询polling（或者叫轮询）的方式读取串口，现在在Linux下面也可以使用事件驱动的方式。



概述

yafeilinux.com以前推出了“Qt串口通信专题教程”，得到很多网友的关注和支持。对此我们表示感谢！由于最近对Wincom和Lincom进行了更新整合，推出了可以跨平台编译的QCom，所以有必要推出本文档。本文档是“Qt串口通信专题教程”的升级和改进。希望通过本文档，可以帮助大家理解和掌握基于QT的串口通信程序的编写。以下是跨平台串口调试助手QCom在Ubuntu下的界面截图。

    本教程分为三部分进行讲述。第一部分讲解关于串口通信的基础知识；第二部分讲解qextserialport类的结构和使用方法；第三部分结合QCom实例具体的讲解qextserialport类的应用。在本教程中我们更注重串口通信相关知识的讲解，而不是界面的设计。关于界面的设计和关于QT更多的知道请关注qter.org的精品文章专区。

第一部分 关于串口通信的基础知识
   
一、连接方式

    目前使用最广泛的串口为DB9接口，它适用于较近距离的通信。一般小于10米。DB9接口有9个针脚。它们的编号名称和作用如下表所示：

    虽然DB9接口有9个针脚，但在做普通的通信用时候，只会用到2、3、5这三个针脚。这也就是我们通常所说的“三线制”连接方法。在连接的时候需要把一方的2针脚接别一方的3针脚，也就是一方的接收针脚连接另一方的发送针脚。5针脚和另一方的5针脚相连。也就是两方的信号地连接在一起。如下图所示：



二、参数简介

    波特率：这是一个衡量通信速度的参数。它表示每秒钟传送的bit的个数。例如9600波特表示每秒钟发送9600个bit。
    数据位：这是衡量通信中实际数据位的参数，当计算机发送一个信息包，实际包含的有效数据位个数。
    停止位：用于表示单个包的最后一位。典型的值为1和2位。由于数据是在传输线上定时的，并且每一个设备有其自己的时钟，很可能在通信中两台设备间出现了小小的不同步。因此停止位不仅仅是表示传输的结束，并且提供计算机校正时钟同步的机会。适用于停止位的位数越多，不同时钟同步的容忍程度越大，但是数据传输率同时也越慢。
    奇偶校验位：它是串口通信中一种检错方式。常用的检错方式有：偶、奇校验。当然没有校验位也是可以的。软件只以设置好校验方式就可以了，真正的校验过程是由硬件自动完成的。编程是不需要考虑。



三、数据传输方式
    关于这一部分的详细介绍，可以参考“串口通讯—异步通信方式”一文。


第二部分 qextserialport类的结构和使用方法

    在Qt中并没有特定的串口控制类，现在大部分人使用的是第三方写的qextserialport类，我们这里也使用了该类。

一、文件下载

官方下载地址：
http://code.google.com/p/qextserialport/wiki/Downloads
在本文中所使用的qextserail的版本为1.2rc。

二、文件内容介绍

1.下载到的文件为qextserialport-1.2rc.zip ，解压并打开后其内容如下。

下面分别介绍：
（1）doc文件夹中的文件内容是QextSerialPort类和QextBaseType的简单的说明，我们可以使用记事本程序将它们打开。
（2）examples文件夹中是几个例子程序，可以看一下它的源码，不过想运行它们好像会出很多问题啊。
（3）src文件夹中是QextSerialPort类的代码，真正使用到的源文件都存这个文件夹中。现在来详细看一下src文件夹下有哪些文件，如下图：

这么多文件，哪真正对我们有用的又有哪些呢？

在windows下需要用到的文件有：
qextserialport.cpp
qextserialport_global.h
qextserialport.h
qextserialport_p.h
qextserialport_win.cpp

在linux下需要的文件有：
qextserialport.cpp
qextserialport_global.h
qextserialport.h
qextserialport_p.h
qextserialport_unix.cpp

仔细看一下，会发现，在两个不同的平台下需要的文件前几个是一样的，只有最后一个不同。这说明，不同平台实现的具体代码存在于这两个文件中。其它的一些文件用不到，可以不用管它。

2.qextserialport的简单介绍
    qextserialport类的声明在qextserialport.h中。我们可以调用的成员函数和一些类型定义都可以在这个文件中找到。下边我们来看一下qextserialport类的定义：

class QEXTSERIALPORT_EXPORT QextSerialPort: public QIODevice

可以看到继承自QIODevice类，所以该类的一些函数我们也可以直接来使用。这个类提供给我们四个构造函数，在实际编程中可以酌情使用。声明如下：

explicit QextSerialPort(QueryMode mode = EventDriven, QObject *parent = 0);
explicit QextSerialPort(const QString &name, QueryMode mode =EventDriven, QObject *parent = 0);
explicit QextSerialPort(const PortSettings &s, QueryMode mode =EventDriven, QObject *parent = 0);
QextSerialPort(const QString &name, const PortSettings &s,QueryMode mode = EventDriven, QObject *parent=0);

关于explicit这个关键字的作用，如果有兴趣可以问一问google，如果没兴趣可以直接忽略它。另外，这个类提供给我们可以调用的关键成员函数有：

void setBaudRate(BaudRateType);//设置波特率
void setDataBits(DataBitsType);//设置数据位
void setParity(ParityType);//设置校验类型
void setStopBits(StopBitsType);//设置停止位

当然还有其它的一些成员函数，详细内容可以打开qextserialport.h这个文件来看一下，我们在这里就不一一列出了。
在QextSerialPort类中还涉及到了一个枚举变量QueryMode。它有两个值Polling和EventDriven 。QueryMode指的是读取串口的方式，下面我们称为查询模式，我们将Polling称为查询方式Polling，将EventDriven称为事件驱动方式。
    事件驱动方式EventDriven就是使用事件处理串口的读取，一旦有数据到来，就会发出readyRead()信号，我们可以关联该信号来读取串口的数据。在事件驱动的方式下，串口的读写是异步的，调用读写函数会立即返回，它们不会冻结调用线程。
而查询方式Polling则不同，读写函数是同步执行的，信号不能工作在这种模式下，而且有些功能也无法实现。我们需要自己建立定时器来读取串口的数据。

需要注意的是：以前版本的QextSerialPort类，在linux下面只能使用Polling查询的方式读取串口，而该类的最新版本已经支持在linux下使用事件驱动的方式读取串口。


第三部分 qextserialport类在QCom跨平台串口调试助手中的应用
   
QCom跨平台调试助手是qter.org推出的串口通信调试软件，源代码完全公开。大家可以完全免费的使用这个软件，并可以通过这个软件学习QT环境下的串口通信编程。（项目主页）

一、在qcom.pro文件中如何实现不同平台下使用不同的源文件

    以下是qcom.pro的部分代码：
SOURCES += main.cpp\
        mainwindow.cpp \
            aboutdialog.cpp\
        qextserial/qextserialport.cpp
HEADERS  += mainwindow.h \
            aboutdialog.h\
       qextserial/qextserialport_global.h \
        qextserial/qextserialport_p.h\
        qextserial/qextserialport.h
win32 {  
//如果是windows环境把qextserialport_win.cpp加入到源文件列表中
        SOURCES+= qextserial/qextserialport_win.cpp
}
unix {
//如果是linux环境把qextserialport_unix.cpp加入到源文件列表中
        SOURCES+= qextserial/qextserialport_unix.cpp
}

二、qextserialport的定义和设置

下面这段代码存在于QCom项目中的mainwindow.cpp文件中：

    #ifdef Q_OS_LINUX
        myCom= new QextSerialPort("/dev/" + portName);
#elif defined (Q_OS_WIN)
            myCom= new QextSerialPort(portName);
#endif
connect(myCom, SIGNAL(readyRead()), this, SLOT(readMyCom()));

可以看到，在不同的平台下，我们给QextSerialPort类的参数是不同的。参考上一部分中的QextSerialPort类的构造函数，可以看到，这里采用的是“事件驱动”方式。
    最后一行代码实现了信号的槽的连接。这样，当串口收到数据以后，就会调readMyCom()函数。
