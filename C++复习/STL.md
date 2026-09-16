# 顺序容器
vector动态数组、deque分段数组、list双向链表、forward_list单向链表、array固定数组。元素按插入顺序排列。核心区别在于内存布局和操作效率：

## 1.vector 本质是动态数组，内部通过三个指针管理
![[Pasted image 20260916113753.png]]
start指向已分配内存的起始位置；finish指向最后一个元素的下一个位置；
finish-start=size（）
end_of_storage指向已分配内存的末尾
end_of_storage-start=capacity()
连续内存，随机访问O(1)，尾部插入均摊O(1)，中间插入O(n);
![[Pasted image 20260916115907.png]]
容量capacity≥大小size，扩容时通常按照2倍（gcc是2倍，MSVC是1.5倍）增长，旧数据被拷贝（C++11移动）到新内存，释放旧内存，更新三个指针

### 容量管理接口：
1.reserve(n):预分配至少n个空间
2.resize(n)：改变元素个数
3.resize(n,val):改变元素个数，新增的用val填充
4.shrink_to_fit：请求capacity缩减到size，释放多余容量
5.clear：情况
上述所有涉及到扩容或降容都会导致迭代器失效，需要重新获取迭代器
### 修改操作：
1.尾部操作：push_back、emplace_back（原地构造避免临时对象）、pop_back
2.中间插入/删除操作（O(n)需移动元素）：insert（pos，val）、
insert（pos、count、val）、insert（pos，{a,b,c}）
erase(pos)删除单个元素、erase(start,end)
erase返回指向下一个有效元素的迭代器，遍历删除时需要注意及时更新iterator

## 2.分段数组 deque
核心设计思想是分段连续存储，由两部分组成：
![[Pasted image 20260916121103.png]]
**缓冲区buffer**：固定大小的连续内存块，真正存储数据；
**中控器map**：一个指针数组，每个元素指向一个缓冲区的首地址
### deque迭代器
包含四个指针：当前元素指针、当前缓存区的起始地址、当前缓冲区的末尾地址、指向map中当前缓冲区的指针

## 3.list 本质带头节点的双向循环链表
### 节点结构
![[Pasted image 20260916122337.png]]
每个节点额外占用前后两个指针，16个字节

### list迭代器
list迭代器是双向迭代器，内部只封装了一个节点指针
![[Pasted image 20260916122621.png]]
可以支持++、--等，但不支持+n、-n无随机访问能力

- **forward_list**：单向链表，比 list 省一个指针的空间，只能单向遍历。

### 核心成员函数
1.splice——零拷贝节点转移
	list独特功能，可以在O(1)时间内将一个节点的ownership转移到另一个list，不发生任何元素的拷贝或移动：
	![[Pasted image 20260916122935.png]]
2.merge——合并两个已排序链表
	![[Pasted image 20260916123038.png]]

### 迭代器失效规则
与vector和deque相比，list迭代器极其稳定
	![[Pasted image 20260916123731.png]]


**总结：**
	**需要频繁访问或者尾部插入/删除元素，选vector；**
	**需要频繁头部插入/删除元素，选deque**
	**节点很大且需要频繁中间增删元素，或需要迭代器的绝对稳定性，选list**
	**编译期可以确定大小，选array**
# 关联容器
**一套内核，四套接口**
![[Pasted image 20260916124254.png]]

### 红黑树内核
#### 1.节点结构
和 list的设计结构相似，采用两层结构，结构指针与数据分离：
![[Pasted image 20260916124422.png]]
基类与数据类型解耦，便于实现通用的树操作；
#### 2.红黑树5条性质
	1.每个节点要么是红，要么是黑；
	2.根结点必须是黑色；
	3.所有叶子结点视为黑色；
	4.红色节点的两个子节点必须是黑色
	5.从任意节点到其所有后代叶子结点的路径中，黑色节点数量相等
	
map/multimap、set/multiset。**底层红黑树，元素自动有序，操作Olog(n)**
map：键值对，键唯一。需要有序遍历或范围查询时用；
set:只存键，自动去重排序。需要判断元素是否存在且要有序时使用；
multimap/multiset：允许重复键
# 无序容器
**底层哈希表，不保证顺序**
unordered_map/unordered_mutimap：键值对，平均O(1)查找、插入。只需要快速查找不需要有序时用；
unordered_set/unordered_,utiset：只存键，平均O(1)判断存在性

| 需求场景         | 推荐容器          |
| ------------ | ------------- |
| 随机访问、尾部增删    | vector        |
| 两端频繁增删(头部)   | deque         |
| 中间频繁增删（已知位置） | list          |
| 有序键值查找/范围查询  | map           |
| 快速查找（不需要有序）  | unordered_map |
| 去重+有序        | set           |
| 去重+快速判断存在    | unordered_set |

# 容器适配器

stack、queue、priority_quieue。不是独立容器，是对底层容器的接口封装。

stack和queue默认用deque，核心原因为：
1.不需要遍历，stack和queue都不提供迭代器；
2.扩容开销小，deque扩容只需新增缓冲区，不搬移数据；
3.两端操作效率高，queue需要push_back和popfront，deque两端都是O(1),vector头部操作是O(n);
4.内存利用率高：相比list，deque不需要每个节点存前后指针，空间浪费更少

# allocator
allocator是STL中负责内存管理的组建，核心作用是把容器的数据结构逻辑和底层内存操作解耦。
有四个职责：
## 1.分配原始内存allocate；
allocate(n):分配能容纳n个对象的原始内存，不调用构造函数；


## 2.释放内存deallocate；
deallocate(p, n):释放之前分配的内存，不调用析构函数


## 3.在已有内存上构造对象（construct）；
construct(p,args...):在已分配的内存地址上用aplacement new构造对象
## 4.析构对象但不释放内存（destroy）
