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


| 层级         | 职责                      | 典型类                   |
| ---------- | ----------------------- | --------------------- |
| Model模型    | 管理数据、提供统一访问接口、通知数据变更    | QAbstractItemModel    |
| View视图     | 负责布局、滚动、选择等界面框架，不直接操作数据 | QTableView、QListView  |
| Delegate委托 | 负责单元格级别的绘制与编辑           | QAbstractItemDelegate |
## MVD三者协作

MVD的三层组件通过信号与槽进行松耦合通信：
1. 数据变更——模型发信号——视图刷新
		模型内部数据变化时，调用beginInsertRows()/endInsertRows等通知宏，发出dataChange、layoutChanged等信号，视图收到后执行局部刷新（不是全量重绘）
2. 用户交互——视图发信号——委托介入
		用户双击单元格时，视图触发编辑事件，委托接管编辑生命周期。
3. 编辑完成——委托发信号——模型更新
		委托通过createEditor创建编辑器、setEditorData载入数据、setModelData将修改回写模型。

### 为什么需要Delegate

在MVC架构中，Controller负责处理用户输入和界面交互；而Qt的MVD中，Delegate主要承担了“如何画”和“如何改”的双重职责：
1. 视觉渲染（paint）：通过重写paint函数，可以绘制进度条、复选框、图标文字等效果
2. 编辑管理：通过createEditor、setEditorData、setModelData、updateEditorGeometry四件套，完整接管编辑生命周期
3. 尺寸计算：告知视觉单元格应占据的宽高
![[Pasted image 20260922005340.png]]

### MVC和MVD区别
![[Pasted image 20260922143400.png]]

# Qt的事件处理机制

Qt的事件处理机制是一套事件循环+三级分发+事件过滤器的完整体系，本质上是“操作系统消息——Qt事件对象——目标对象处理”的逐层转换分发流程

## 1.事件循环
Qt程序启动后，QApplication::exec()会进入一个无限循环，持续的做三件事：
1. 从队列取事件；
2. 分发给目标对象
3. 无事件时休眠，避免CPU空转
![[Pasted image 20260922012642.png]]

事件来源包括

| 事件来源 | 示例                   |
| ---- | -------------------- |
| 操作系统 | 鼠标点击、键盘输入、窗口拖拽、显示器插拔 |
| Qt内部 | 定时器超时、窗口重绘请求         |
| 应用程序 | postEvent、sendEvent  |

## 2.事件投递：同步or异步

Qt提供两种事件投递方式：

| 方式  | 函数                            | 行为           | 适用场景       |
| --- | ----------------------------- | ------------ | ---------- |
| 同步  | QCoreApplication::sendEvent() | 立即阻塞处理，不入队   | 高优先级须立即响应  |
| 异步  | QCoreApplication::postEvent() | 放入事件队列，非阻塞返回 | 跨线程通信、重绘合并 |
postEvent支持事件压缩，短时间多次update产生的重绘事件会被合并

## 3.三级分发链路：从队列到处理函数

当事件到达目标对象时，Qt按以下顺序逐层分发：
![[Pasted image 20260922013204.png]]
### 3.1 事件过滤器EventFilter
允许一个对象拦截发送给另一个对象的事件，无需子类化目标控件：
![[Pasted image 20260922013532.png]]
返回true——事件被拦截，目标对象收不到
返回false——事件继续向下传递
一个对象可以安装多个过滤器，后安装的先执行

### 3.2 event()函数
event() 函数是目标对象的通用事件入口，内部是巨大的Switch分发器，根据事件类型调用对应的处理器：
![[Pasted image 20260922013807.png]]
适用场景：需要同事处理多种事件类型，或在事件分发前做统一预处理，比如快捷键

### 3.3 特定事件处理器

直接重写对应的时间处理函数：
![[Pasted image 20260922013924.png]]

## 4.事件的接受与传播
每个事件都有accept和ignore标记

accept：该事件已经处理，停止向父对象传播
ignore：事件未完全处理，向父对象回溯
![[Pasted image 20260922014125.png]]
默认行为是accept，如果重写了时间处理函数但是既没调用accept也没调ignore，Qt默认认为事件被处理accept，不会再向父对象传播。

当子控件ignore后，事件向父对象传播时，不是简单调用父控件的对应事件处理函数如**`mousePressEvent`** 而是重启一轮完整的事件派送流程：
父控件的 eventFilter() → 父控件的 event() → 父控件的 mousePressEvent()
## 5.嵌套事件循环

Qt支持在事件循环中启动另一个循环，典型场景是模态对话框：
![[Pasted image 20260922014232.png]]
dialog.exe()内部新建一个QEventLoop
父窗口的事件循环暂时被挂起，但对话框自己的循环正常处理事件
对话框关闭时，内存循环退出，控制权返回父窗口

哪些场景会手动创建 `QEventLoop` ？
1. 模态对话框（Qt 内部实现）
2. 同步等待异步操作，如：网络请求、等待某个信号触发
		![[Pasted image 20260922015010.png]]![[Pasted image 20260922015024.png]]

3. 分段式耗时任务：大数据处理时，每处理一批数据就释放一次事件队列，让界面有机会刷新


### 6. processEvents

作用是手动触发一次当前线程的事件队列迭代——把积压的待处理事件一次性处理

它的"不可控"体现在三个层面：**递归重入、时序错乱、状态撕裂**