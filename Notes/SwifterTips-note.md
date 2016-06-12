# Swfiter Tips

## [Delegate](http://swifter.tips/delegate/)

1. Swift 的 protocol 是可以被除了 class 以外的其他类型遵守的
2. struct 或是 enum 这样的类型，本身就不通过引用计数来管理内存，所以也不可能用 weak 这样的 ARC 的概念来进行修饰
3. 通过在 protocol 前面加上 @objc 关键字来将 protocol 声明为 Objective-C 的
4. 在 protocol 声明的名字后面加上 class，这可以为编译器显式地指明这个 protocol 只能由 class 来实现