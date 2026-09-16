顺序容器
	vector动态数组、deque分段数组、list双向链表、forward_list单向链表、array固定数组。元素按插入顺序排列
	核心区别在于内存布局和操作效率：
	1.vector:本质是动态数组，内部通过三个指针管理
		![[Pasted image 20260916113753.png]]
		start指向已分配内存的起始位置；finish指向最后一个元素的下一个位置；
		finish-start=size（）
		end_of_storage指向已分配内存的末尾
		end_of_storage-start=capacity()
		连续内存，随机访问O(1)，尾部插入均摊O(1)，中间插入O(n);
		容量capacity≥大小size，扩容时通常按照2倍（gcc是2倍，MSVC是1.5倍）增长，旧数据被拷贝（C++11移动）到新内存，释放旧内存，更新三个指针

		容量管理接口：
			1.reserve(n):预分配至少n个空间
			2.resize(n)：改变元素个数
			3.resize(n,val):改变元素个数，新增的用val填充
			4.shrink_to_fit：请求capacity缩减到size
			5.clear：情况
			上述所有涉及到扩容或降容都会导致迭代器失效
	2.
关联容器
	map/multimap、set/multiset。底层红黑树，元素自动有序，操作Olog(n)

无序容器
	unordered_map/unordered_mutimap、unordered_set/unordered_,utiset。底层哈希表，无序，平均O(1)

容器适配器
	stack、queue、priority_quieue。不是独立容器，是对底层容器的接口封装。