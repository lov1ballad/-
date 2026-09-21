# Qt的元系统
Qt的元对象系统（MOS）是Qt框架最核心的基础，本质上是一套编译期代码生成+运行时静态数据分发的反射机制，为C++提供了标准语言不具备的运行时能力。

## 1.三大支柱
### QObject基类
所有需要元对象能力的类必须直接或者间接继承，提供基础设施

### Q_OBJECT宏
在类中声明后启用元对象功能：信号、槽、动态属性等
![[Pasted image 20260922000654.png]]
这些被插入的函数只有声明没有实现——实现由moc在生成的moc.cpp中补全

### MOC元对象编译器
Qt自带的代码生成工具，在常规编译前扫描头文件，生成包含元对象代码的moc.cpp文件

## 2.元对象系统的核心功能

1. 运行时类型识别
2. 动态属性系统setProperty
3. 动态方法调用，Q_INVOKABLE ，QMetaObject::inockeMethod
4. 不依赖平台

# Qt的信号槽实现原理

Qt的信号槽机制是Qt框架最核心的特性，其本质是一套类型安全的观察者模式，通过元对象系统（meta-object-system）实现对象间的松耦合通信。
完整实现分为三个阶段：
## 1.编译期：moc代码生成

所有使用信号槽的类必须继承QObject并在类声明中包含Q_OBJECT宏。
**元对象编译器MOC**(meta-object-compiler)作为预处理器，在编译前扫描头文件，为每个含Q_OBJECT宏的类生成一个moc_*** . cpp文件，其中包含：
### moc.cpp文件包含信息：
1.**元数据表**：一个静态元对象实例：staticMetaObject。 记录类名、信号/槽的索引、参数类型ID、父类指针、方法列表、属性信息等；
2.**信号函数的实现体**：比如在头文件中只声明了信号(如void valueChange(int))，没有写函数体（正常也不会写）——moc会帮着补上。生成的代码大致是：![[Pasted image 20260921143053.png]]
信号函数本身不包含任何业务逻辑，只负责将参数打包为void* 数组，然后调用QMetaObject::activate()进行分发。
3.qt_metacall()虚函数重写：一个巨大的switch语句，根据方法索引动态调用对应的槽函数。

## 2.连接建立

调用QObject::connect()时
connect(sender, &Sender::valueChanged, receiver, &Receiver::onValueChanged);
1. **提取信号和槽的索引**：通过元对象系统查找信号和槽在各自类中的方法索引；
2. **编译期类型检查**：通过**QtPrivate::FunctionPointer**提取函数签名，用static_assert校验参数兼容性，参数不匹配直接编译报错；
3. **创建Connection对象**：QT内部QObjectPrivate::**Connection对象**，记录发送者、信号索引、接收者、槽索引、连接类型，并将这个对象加入到发送者的**connectionLists**(一个以信号索引为下标的数组，每个信号挂载一个链表)。

## 3.运行时：信号触发与槽调用

当执行emit valueChanged(42)时，实际调用的是元对象编译器moc生成的信号函数，最终进入QMetaObject::activate()；
### 执行流程
1. 根据**信号索引**从**connectionLists**中取出对应的**连接链表**；
2. **遍历链表**，对每个连接检查接收者是否存活、连接类型是什么；
3. **根据连接类型决定调用方式**：
	....DirectConnection：在同一线程直接通过qt_metacall()同步调用槽函数，无堆分配，性能接近普通函数调用；
	
	.....QueuedConnection:跨线程时，将信号参数深拷贝后封装为**QMetaCallEvent**，通过QCoreApplication::postEvent()投递到**接收者所属线程**的事件队列，由事件循环异步处理，目标线程的事件循环取出事件，在目标线程中执行槽函数
	
	....AutoConnection：默认参数，同线程等同于Direct，跨线程等同于Queued。
	
	....BlockingQueuedConnection：跨线程时投递事件后阻塞原发送线程，直到槽函数执行完毕

跨线程通信的究其根本原因是，QObject内部有一个threadData指针，该指针指向目标线程的事件循环数据结构，如果sender的threadData和receiver的一致，则通过直连发送响应槽函数，否则通过事件队列
## 此设计思想的好处
1. 松耦合：发送者不知道也不需要关心谁接收了信号，接收者也不知道信号来自哪里；
2. 类型安全：信号与槽的参数签名必须兼容（槽的参数可以少于信号，多余的参数被忽略）；
3. 一对多/多对一：一个信号可以连接多个槽，多个信号也可以连接同一个槽；
4. 线程安全：通过事件队列机制，天然支持跨线程通信（QueuedConnection）；
5. 自动清理：当QObject被销毁时，与其相关的所有连接自动断开，避免野指针


# Qt中的mvd

Model——View——Delegate，模型视图委托。是Qt框架中用于实现数据与界面分离的核心设计架构，本质是经典MVC模式在GUI场景下的演进——Qt将Controller的职责融入到Delegate中


| 层级         | 职责                   | 典型类 |
| ---------- | -------------------- | --- |
| Model模型    | 管理数据、提供统一访问接口、通知数据变更 |     |
| View视图     |                      |     |
| Delegate委托 |                      |     |
