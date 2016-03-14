原文地址：<http://blog.sunnyxx.com/2014/04/30/ios_iterator/>

1. ios中常用的遍历运算方法

	- 经典for循环
	- for in (NSFastEnumeration)，若不熟悉可以参考《nshipster介绍NSFastEnumeration的文章》
	- makeObjectsPerformSelector
	- kvc集合运算符
	- enumerateObjectsUsingBlock
	- enumerateObjectsWithOptions(NSEnumerationConcurrent)
	- dispatch_apply

2. 实验

	- 实验条件，实验使用CFAbsoluteTimeGetCurrent()记录时间戳来计算运行时间，单位秒。运行在iphone5真机（双核cpu）
		- 分别使用有100个对象和1000000个对象的NSArray，只取对象，不执行操作，测试遍历速度
		- 使用有100个对象的NSArray遍历执行doSomethingSlow方法，测试遍历中多任务运行速度
	- 100 对象遍历操作：

	~~~
	经典for循环 - 0.001355
	for in (NSFastEnumeration) - 0.002308
	makeObjectsPerformSelector - 0.001120
	kvc集合运算符(@sum.number) - 0.004272 
	enumerateObjectsUsingBlock - 0.001145
	enumerateObjectsWithOptions(NSEnumerationConcurrent) - 0.001605
	dispatch_apply(Concurrent) - 0.001380
	~~~
	- 1000000 对象遍历操作：

	~~~
	经典for循环 - 1.246721
	for in (NSFastEnumeration) - 0.025955
	makeObjectsPerformSelector - 0.068234
	kvc集合运算符(@sum.number) - 21.677246
	enumerateObjectsUsingBlock - 0.586034
	enumerateObjectsWithOptions(NSEnumerationConcurrent) - 0.722548
	dispatch_apply(Concurrent) - 0.607100
	~~~
	- 100 对象遍历执行一个很费时的操作：

	~~~
	经典for循环 - 1.106567
	for in (NSFastEnumeration) - 1.102643
	makeObjectsPerformSelector - 1.103965
	kvc集合运算符(@sum.number) - N/A
	enumerateObjectsUsingBlock - 1.104888
	enumerateObjectsWithOptions(NSEnumerationConcurrent) - 0.554670
	dispatch_apply(Concurrent) - 0.554858
	~~~

	- 值得注意的
		- 对于集合中对象数很多的情况下，for in (NSFastEnumeration)的遍历速度非常之快，但小规模的遍历并不明显（还没普通for循环快）
		- 使用kvc集合运算符运算很大规模的集合时，效率明显下降（100万的数组离谱的21秒多），同时占用了大量内存和cpu
		- enumerateObjectsWithOptions(NSEnumerationConcurrent)和dispatch_apply(Concurrent)的遍历执行可以利用到多核cpu的优势（实验中在双核cpu上效率基本上x2）

3. 遍历实践Tips

	- 倒序遍历
		- `NSArray`和`NSOrderedSet`都支持使用`reverseObjectEnumerator`倒序遍历
		- 使用`enumerateObjectsWithOptions:NSEnumerationReverse`也可以实现倒序遍历
	- 使用block同时遍历字典key，value
	- 对于耗时且顺序无关的遍历，使用并发版本
	- 代码可读性和效率的权衡