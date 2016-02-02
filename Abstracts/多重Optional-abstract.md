原文链接：http://swifter.tips/multiple-optional/

1. 我们使用的类型后加上 ? 的语法只不过是 Optional 类型的语法糖
2. 可以在 Optional 中装入任意东西的，甚至也包括 Optional 对象自身
3. 将一个非nil的Optional赋值给多重Optional与直接赋值字面值给多重Optional是等效的，如`var string: String? = "string"` `var anotherString: String?? = string`与`var literalOptional: String?? = "string"`，根据类型判断，只能将Optional(String)放入到literalOptional，因此可以猜测到anotherStrig与literalOptional是等效的。
4. 赋值nil时与上面情况不同
	- `var aNil: String? = nil`，将aNil赋值给多重Optional时，是将一个包装了nil的Optional放入Optional中
	- `var literalNil: String?? = nil`，将nil直接赋值给多重Optional，是将nil放入Optional中，对literNil解包出来的是nil，`if let b = literalNil {...}`中的代码不会执行
	- 在用 lldb 进行调试时，直接使用 po 指令打印 Optional 值的话，为了看起来方便，lldb 会将要打印的 Optional 进行展开。如果我们直接打印上面的 anotherNil 和 literalNil，得到的结果都是 nil
	- 可以使用 fr v -R 命令来打印出变量的未加工过时的信息
