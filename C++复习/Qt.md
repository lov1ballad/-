# Qt的信号槽实现原理

Qt的信号槽机制是Qt框架最核心的特性，其本质是一套类型安全的观察者模式，通过源对象系统（meta-object-system）实现对象间的松耦合通信。
完整实现分为三个阶段：
## 1.编译期：moc代码生成

所有使用信号槽的类必须继承QObject并在类声明中包含Q_OBJECT宏。
MOC(meta-object-compiler)作为预处理器，在编译前扫描头文件，为每个含Q_OBJECT宏的类生成一个moc_*** . cpp文件，其中包含：
### moc.cpp文件包含信息：
1.元数据表：记录类名、信号/槽的索引、参数类型ID、属性信息等；
2.信号函数的实现体：比如在头文件中只声明了信号(如void valueChange(int))，没有写函数体（正常也不会写）——moc会帮着补上。生成的代码大致是：![[Pasted image 20260921143053.png]]
信号函数本身不包含任何业务逻辑，只负责将参数打包为void* 数组，然后调用QMetaObject::activate()进行分发。
3.qt_metacall()虚函数重写：一个巨大的switch语句，根据方法索引动态调用对应的槽函数。

## 2.运行时：连接建立

调用QObject::connect()时，QT内部会创建一个QObjectPrivate::Connection对象，记录发送者、信号索引、接收者、槽索引和连接类型，并将其加入发送者connectionLists(一个以信号索引为下标的稀疏数组，每个信号挂载一个连接链表)。
connect(sender, &Sender::valueChanged, receiver, &Receiver::onValueChanged);

这种方式在**编译期**会通过QtPrivate::FunctionPointer提取函数签名，用static_assert做类型检查，参数不匹配直接编译报错。


## 3.运行时：信号触发与槽调用

当执行emit valueChanged(42)时，实际调用的是moc生成的信号函数，最终进入QMetaObject::activate()；
### 执行流程
1. 根据信号索引从connectionLists中取出对应的连接链表；
2. 遍历链表，对每个连接检查接收者是否存活、连接类型是什么；
3. 根据连接类型决定调用方式：
DirectConnection：在同一线程直接通过qt_metacall()同步调用槽函数，无堆分配，性能接近普通函数调用；