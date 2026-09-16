顺序容器
	vector动态数组、deque分段数组、list双向链表、forward_list单向链表、array固定数组。元素按插入顺序排列

关联容器
	map/multimap、set/multiset。底层红黑树，元素自动有序，操作Olog(n)

无序容器
	unordered_map/unordered_mutimap、unordered_set/unordered_,utiset。底层哈希表，无序，平均O(1)

容器适配器
	stack、queue、priority_quieue。不是独立容器，是对底层容器的接口封装。