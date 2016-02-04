原文链接：[http://ningandjiao.iteye.com/blog/2226688](http://ningandjiao.iteye.com/blog/2226688)

1. Self Requirement
	- 主要的作用就是解决使用Protocol（接口）之后，类型缺失的问题，即，当在对象类型上使用了protocol类型之后，如果还需要调用对象自有的一些方法，那么就不得不用as！做一个强制类型转换

		```
		protocol Ordered {
			func precedes(other: Self) -> Bool
		}
		```
	- 一旦在protocol中使用了Self，那么该protocol就不能做类型限定符了，即不能再使用该类型指定参数，属性的类型，其只能用于泛型中
	- 通过这个特性，可以让代码更加清晰，隔离性更好，处理Number时，只需要在意Number而无需去考虑其它的实现了Ordered的对象。同时，因为整个程序运行时的类型都是固定的，对编译器优化程序也有很大的帮助
2. Protocol Extension
	- 使用接口时，经常会出现如下的场景，几个类实现了同一个接口，接口的其中一个方法，在几个类中实现都是一样的
	- 这个时候就会出现几个类有重复代码的情形，一般有2种方式来消除掉这种重复
		- 一是给这几个毫无关系的类加一个共同父类来做代码共享（强烈建议不要这样做）
		- 二是把这个方法抽出到一个单独的类中，再通过组合方式把功能引回需要使用的类（比较麻烦）
		- protocol extension就完美的解决了这样的问题
	- Protocol Extension还提供了一个有趣的特性，限制扩展支持的类型
		- 可以在实现Protocol Extension时，直接指定必须满足某种条件才能使用该扩展中的方法，前面提到的是子类决定要不要接受Protocol Extensio中的默认实现，而这个特性是Protocol Extension设置默认实现的使用门槛

