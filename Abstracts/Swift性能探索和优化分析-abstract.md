原文链接：<http://onevcat.com/2016/02/swift-performance/>

此文为[喵神](http://onvcat.com)的 [Swift 性能探索和优化分析](http://onevcat.com/2016/02/swift-performance/)的摘录，摘抄下自己认为重要的点，方便学习。

# 为什么 Swift 的性能值得期待
1. Swift 在编译期间就完成了方法的绑定，在方法调用上不再类似 Smalltalk 的消息发送，而是直接获取方法地址并进行调用。
2. Objective-C 对运行时查找方法的过程进行了缓存和大量的优化。
3. Swift 的运行时和代码编译期间的类型是一致的，这样编译器可以得到足够的信息来在生成中间码和机器码时进行优化。
4. Swift 的编译过程比 Objective-C 多一个环节——生成 Swift 中间代码（Swift Intermediate Language，SIL）。SIL 中包含有很多根据类型假定的转换，这为之后进一步在更底层级优化提供了良好的基础。
5. Swift 具有良好的内存使用的策略和结构。Swift 巧妙的规避了不必要的值类型复制，而仅在必要时进行内存分配。

# 对性能进行测试
1. 在 Cocoa 中我们可以使用 `CACurrentMediaTime` 来获取精确的时间。这个方法将会调用 mach 底层的 mach_absolute_time()，它的返回是一个基于 Mach absolute time unit 的数字，我们通过在方法调用前后分别获取两次时刻，并计算它们的间隔，就可以了解方法的执行时间。
2. 使用 Instruments 的 Time Profiler 来在更高层面寻找代码的性能弱点。
3. 在 Xcode 中默认的测试框架 XCTest 提供了检测并汇报性能的方法：`measureBlock`。

	~~~swift
	func testPerformance() {
	    measureBlock() {
	        // 需要性能测试的代码
	    }
	}
	~~~
	
# 优化手段，常见误用及对策
## 多线程、算法及数据结构优化
1. 对于影响了 UI 流畅度的代码，我们可以将其放到后台线程进行运行.
2. 进一步地，我们可以考虑改进算法和使用的数据结构来提高效率.
3. 对于重复的工作，合理地利用缓存的方式可以极大提高效率，这是在优化时可以优先考虑的方式。Cocoa 开发中 `NSCache` 是专门用来管理缓存的一个类，合理地使用和配置 `NSCache` 把开发者中从管理缓存存储和失效的工作中解放出来。
4. 在程序开发时，数据结构使用上的选择也是重要的一环。
5. 如果项目中有很多数学计算方面的工作导致了效率问题的话，考虑并行计算能极大改善程序性能。
6. iOS 和 OS X 都有针对数学或者图形计算等数字信号处理方面进行了专门优化的框架：[Accelerate.framework](https://developer.apple.com/library/tvos/documentation/Accelerate/Reference/AccelerateFWRef/index.html)，利用相关的 API，我们可以轻松快速地完成很多经典的数字或者图像处理问题。
7. [Surge](https://github.com/mattt/Surge)，它是一个基于 Accelerate 框架的 Swift 项目

## 编译器优化
1. Swift 编译器十分智能，它能在编译期间帮助我们移除不需要的代码，或者将某些方法进行内联 (inline) 处理。
2. 编译器优化的强度可以在编译时通过参数进行控制。
3. Xcode 工程默认情况下有 Debug 和 Release 两种编译配置，在 Debug 模式下，LLVM Code Generation 和 Swift Code Generation 都不开启优化，这能保证编译速度。而在 Release 模式下，LLVM 默认使用 "Fastest, Smallest [-Os]"，Swift Compiler 默认使用 "Fast [-O]"，作为优化级别。
4. 我们另外还有几个额外的优化级别可以选择，优化级别越高，编译器对于源码的改动幅度和开启的优化力度也就越大，同时编译期间消耗的时间也就越多。
5. 一些优化等级采用的是激进的优化策略，而禁用了一些检查。这可能在源码很复杂的情况下导致潜在的错误。
6. 如果你使用了很高的优化级别，请再三测试 Release 和 Debug 条件下程序运行的逻辑，以防止编译器优化所带来的问题。
7. Swift 编译器有一个很有用的优化等级："Fast, Whole Module Optimization"，也即 -O -whole-module-optimization。在这个优化等级下，Swift 编译器将会同时考虑整个 module 中所有源码的情况，并将那些没有被继承和重载的类型和方法标记为 final，这将尽可能地避免动态派发的调用，或者甚至将方法进行内联处理以加速运行。开启这个额外的优化将会大幅增加编译时间，所以应该只在应用要发布的时候打开这个选项。

## 尽量使用 Swift 类型
1. 很多 Swift 标准库类型和对应的 Cocoa 类型是可以隐式的类型转换的，但是这个过程却不是免费的。在转换次数很多的时候，这往往会成为性能的瓶颈。
2. 这个转换在一次性处理成千上万条 JSON 数据时会带来严重的性能退化。如果存在这样的情况，一种处理方式是避免使用 Swift 的字典类型，而使用 `NSDictionary`。适当地使用 lazy 加载的方法，也是避免一次性进行过多的类型转换的好思路。
3. 尽可能避免混合地使用 Swift 类型和 NSObject 子类，会对性能的提高有所帮助。

## 避免无意义的 log，保持好的编码习惯
1. Swift 编译器并不会帮我们将 print 或者 debugPrint 删去，在最终 app 中它们会把内容输出到终端，造成性能的损失。
2. 通过添加条件编译来将这些语句排除在 Release 版本外。在 Xcode 的 Build Setting 中，在 **Other Swift flags** 的 Debug 栏中加入 `-D DEBUG` 即可加入一个编译标识。
3. 如果你传入 `print` 的 items 是一个表达式而不是直接的变量的话，这个表达式还是会被先执行求值的。
3. 最好用 `@autoclosure` 修饰参数来重新包装 `print`。这可以将求值运行推迟到方法内部，这样在 Release 时这个求值也会被一并去掉。

# 小结
1. Swift 还是一门很新的语言，并且处于高速发展中。因为现在 Swift 只用于 Cocoa 开发，因此它和 Cocoa 框架还有着千丝万缕的联系。很多时候由于这些原因，我们对于 Swift 性能的评估并不公正。
2. 这门语言本身设计就是以高性能为考量的，而随着 Swift 的开源和进一步的进化，以及配套框架的全面重写，相信在语言层面上我们能获得更好的性能和编译器的支持。
3. 最好的优化就是不用优化。在软件开发中，保证书写正确简洁的代码，在项目开始阶段就注意可能存在的性能缺陷，将可扩展性的考虑纳入软件构建中，按照实际需求进行优化，不要陷入为了优化而优化的怪圈，这些往往都可以让我们避免额外的优化时间，让我们的工作得更加愉快。

# 参考
- [Swift Intermediate Language](http://llvm.org/devmtg/2015-10/slides/GroffLattner-SILHighLevelIR.pdf)
- [NSCache - NSHipster](http://nshipster.com/nscache/)
- [NSCache 文档](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/NSCache_Class/)
- [Surge](https://github.com/mattt/Surge)