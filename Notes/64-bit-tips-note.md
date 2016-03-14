原文链接：<http://blog.sunnyxx.com/2014/12/20/64-bit-tips/>

1. 拒绝基本数据类型和隐式转换
	- int -> NSInteger
	- unsigned -> NSUInteger
	- float -> CGFloat
	- 动画时间 -> NSTimeInterval
	- 如下代码 for 循环一次都没进

	~~~objc
	for (int i = -1; i < items.count; i++) {}
	~~~

2. 使用新版枚举
3. 替代Format字符串
	- 32-bit下，BOOL被定义为`signed char`，@encode(BOOL)的结果是`'c'`  
	- 64-bit下，BOOL被定义为`bool`，@encode(BOOL)结果是`'B'`
	- 32-bit版本的BOOL包括了256个值的可能性，还会引起一些坑，像[这篇文章](http://www.bignerdranch.com/blog/bools-sharp-corners/)所说的。而64-bit下只有0（NO），1（YES）两种可能，终于给BOOL正了名。
4. 不直接取isa指针
	- 编译器已经默认禁用了这种使用，isa指针在32位下是Class的地址，但在64位下利用bits mask才能取出来真正的地址，若真需要，使用runtime的`object_getClass` 和`object_setClass`方法。关于64位下isa的讲解可以看[这篇文章](http://www.sealiesoftware.com/blog/archive/2013/09/24/objc_explain_Non-pointer_isa.html)
5. 解决第三方lib依赖和lipo命令
	- 以源码形式出现在工程中的第三方lib，只要把target加上`arm64`编译就好了。
	- 恶心的就是直接拖进工程的那些静态库(.a)或者framework，就需要重新找支持64-bit的包了。这时候就能看出哪些是已无人维护的lib了，是时候找个替代品了
6. 打印Mach-O文件支持的架构
	- 使用`lipo -info`命令
	- 想看的更详细的信息可以使用`lipo -detailed_info`
	- 还可以使用`file`命令
7. 合并多个架构的包：

	~~~shell
	lipo -create MyLib-32.a MyLib-64.a -output MyLib.a
	~~~
	
8. 支持64-bit后程序包会变大，支持64-bit后，多了一个arm64架构，理论上每个架构一套指令，但相比原来会大多少还不好说
9. 一个lib包含了很多的架构，不会打到最后的包里，如果lib中有armv7, armv7s, arm64, i386架构，而target architecture选择了armv7s, arm64，那么只会从lib中link指定的这两个架构的二进制代码，其他架构下的代码不会link到最终可执行文件中；反过来，一个lib需要在模拟器环境中正常link，也得包含i386架构的指令
10. 官方文档中的注意点：
	- 不要将指针强转成整数
	- 程序各处使用统一的数据类型
	- 对不同类型的整数做运算时一定要注意
	- 需要定长变量时，使用如int32_t, int64_t这种定长类型
	- 使用malloc时，不要写死size
	- 使用能同时适配两个架构的格式化字符串
	- 注意函数和函数指针（类型转换和可变参数）
	- 不要直接访问Objective-C的指针（isa）
	- 使用内建的同步原语（Primitives）
	- 不要硬编码虚存页大小
	- [Go Position Independent](https://developer.apple.com/library/ios/qa/qa1788/_index.html#//apple_ref/doc/uid/DTS40013354)