# 如何在 Swift 中高效地使用 Method Swizzling 笔记

原文地址：<http://swift.gg/2016/03/29/effective-method-swizzling-with-swift/>

1. 在 runtime 执行这类更改时，不能在编译时利用那些可用的安全检查。
2. Swift 关于方法派发是使用静态方法的。
3. 在 Swift 中尽量少用 Method Swizzling，只有当你的问题不能用 Swift 的方式解决，也不能用子类、协议或扩展解决时才使用。
4. `+load` 方法只在 Objective-C 里有，不能在 Swift 里重载。在 Swift 中适合执行 swizzle 最好的地方是 `+initialize`，这是调用类中第一个方法前会调用的方法。所有对修改方法执行的操作需要放在 `dispatch_once` 中，确保这个过程只执行一次。
5. 包含 swizzle 方法的类需要继承自 NSObject。
6. 需要 swizzle 的方法必须要有申明为 dynamic。
7. 用 @objc 将你的 Swift API 暴露给 Objective-C runtime 时，不能确保属性、方法、初始器的动态派发，因为 Swift 编译器可能为了优化代码而绕过 Objective-C tuntime。而将其申明为 dynamic 之后，对其的访问就总会动态派发了。
8. 假如 Swift 编译器将方法的实现内联（inline）了或者访问去虚拟化（devirtualize）了，这个交换过来的新的实现就无效了。（注：这一点并不能很好的理解）
9. 如果想要替换的方法没有声明为 dynamic 的话，就不能 swizzle。（Cocoa 里的都是 dynamic 的？）