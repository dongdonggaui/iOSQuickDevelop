原文链接：[http://onevcat.com/2016/01/create-framework/](http://onevcat.com/2016/01/create-framework/)

1. 什么是静态库 (Static Library)
	- 一系列从源码编译的目标文件的集合，它是你的源码的实现所对应的二进制
	- 配合上公共的 .h 文件，我们可以获取到 .a 中暴露的方法或者成员等
	- 在最后编译 app 的时候.a 将被链接到最终的可执行文件中，之后每次都随着app的可执行二进制文件一同加载，你不能控制加载的方式和时机，所以称为静态库
2. 什么是动态框架 (Dynamic Framework)
	- Framework 其实是一个 bundle，或者说是一个特殊的文件夹
	- 系统的 framework 是存在于系统内部，而不会打包进 app 中
	- app 的启动的时候会检查所需要的动态框架是否已经加载
	- 像 UIKit 之类的常用系统框架一般已经在内存中，就不需要再次加载
3. Universal Framework
	- iOS 8 之前也有一些第三方库提供 .framework 文件，但是它们实质上都是静态库，只不过通过一些方法进行了包装
4. Library v.s. Framework
	- 静态库不能包含像 xib 文件，图片这样的资源文件，其他开发者必须将它们复制到 app 的 main bundle 中才能使用，维护和更新非常困难，而 framework 则可以将资源文件包含在自己的 bundle 中（静态库也可以将这些资源文件放在自己的bundle里吧）
	- 静态库必须打包到二进制文件中，随着 iOS 扩展（比如通知中心扩展或者 Action 扩展）开发的出现，你现在可能需要将同一个 .a 包含在 app 本体以及扩展的二进制文件中
	- 静态库只能随应用 binary 一起加载，而动态框架加载到内存后就不需要再次加载，二次启动速度加快。另外，使用时也可以控制加载时机
5. Cocoa Touch Framework
	- app 中的使用的 Cocoa Touch Framework 在打包和提交 app 时会被放到 app bundle 中，运行在沙盒里，而不是系统中，不同的 app 就算使用了同样的 framework，但还是会有多份的框架被分别签名，打包和加载
	- Cocoa Touch Framework 的推出主要是为了解决两个问题
		- 从 iOS 8 开始的扩展开发
		- Swift，在 Swift 开源之前，它是不支持编译为静态库的
	- Swift runtime 不在系统中，而是打包在各个 app 里的
	- 如果要使用 Swift 静态框架，由于 ABI 不兼容，所以我们将不得不在静态包中再包含一次 runtime，可能导致同一个 app 包中包括多个版本的运行时
6. 包和依赖管理
	- CocoaPods
		- 框架的提供者通过编写合适的 PodSpec 文件来提供框架的基本信息，包括仓库地址，需要编译的文件，依赖等
		- 用户使用 Podfile 文件指定想要使用的框架
		- CocoaPods 会创建一个新的工程来管理这些框架和它们的依赖，并把所有这些框架编译到成一个静态的 libPod.a。然后新建一个 workspace 包含你原来的项目和这个新的框架项目，最后在原来的项目中使用这个 libPods.a
		- 这是一种“侵入式”的集成方式，它会修改你的项目配置和结构
		- 从 0.36.0 开始，可以通过在 Podfile 中添加 `use_frameworks!` 来编译 CocoaTouch Framework
		- `use_frameworks!` 是一个 none or all 的更改，你无法指定某几个框架编译为动态，某几个编译为静态（假设 Pod A 是动态框架，Pod B 是静态，Pod A 依赖 Pod B。要是 app 也依赖 Pod B：那么要么 Pod A 在 link 的时候找不到 Pod B 的符号，要么 A 和 app 都包含 B）
	- Carthage
		- 只支持动态框架
		- 它会将第三方项目 clone 到本地并将对应的 Cocoa Framework target 进行构建
		- 需要自行将构建好的 framework 添加到项目中
		- 去中心化，它直接从 git 仓库获取项目，而不需要依靠 podspec 类似的文件来管理
		- 通过一个文件 `Cartfile` 来指定依赖关系
		- 使用时，需要将用到的框架 Embedded Binary 的方式链接到希望的 App target 中
	- Swift Package Manager
		- 现在暂时不支持 iOS 和 tvOS
		- 通过 `llbuild` （low level build system）的跨平台编译工具将 Swift 编译为 .a 静态库
7. 创建框架
	- framework target 模板，添加源文件，编写代码，编译，完成
	- API 设计
		- 最小化原则，一开始可以提供尽可能少的接口来完成必要的任务，随着逐步的开发和框架使用场景的扩展，可以添加公共接口或者将原来的 internal 或者 private 接口标记为 public 供外界使用
		- 命名考虑
			- 尽量表意清晰，不用奇怪的缩写，如 remove 和 removeAt 、fetch 和 recursivelyFetch 对比
			- 动词或者动词短语开头，而属性名应该是名词
			- 避免名动皆可的词语，比如把 displayName 换为 screenName
			- 如果用一句话无法将一个方法的内容表述清楚的话，这往往就意味着 API 的名字有改进的余地
			- Apple 官方给出了一个很详细的[指南](https://swift.org/documentation/api-design-guidelines/) (Swift API Design Guidelines)
		- 优先测试，测试驱动开发
			- Swift 2 开始可以使用 @testable 来把框架引入到测试 module，这样就可以调用 internal 方法
8. 开发时的选择
	- 命名冲突
		- Swift 中可以通过 module 来提供类似命名空间隔离
		- 在对系统已有的类添加 extension 时，必须添加前缀，NSObject 的 extension 实际上还是依赖于 Objective-C runtime 的，两个拥有相同方法名的框架都在 app 启动时候被加载，运行时究竟调用了哪个方法和加载顺序相关，并不确定
	- 资源 bundle
		- framework 的一大优势是可以在自己的 bundle 中包含资源文件
		- 不需要关心框架的用户的环境，直接访问自己的类型的 bundle 就可以获取框架内的资源
9. 发布框架
	- 选择依赖工具
		- CocoaPods
			- 使用 CocoaPods 提供的命令行工具（`pod spec create MyFramework`）可以创建一个 podspec 模板，按照项目的情况编辑这个文件。关于更详细的用法，可以参看 CocoaPods 的[文档](https://guides.cocoapods.org/making/getting-setup-with-trunk.html)
			- 使用它们的命令行工具来检查 podspec 语法和项目是否正常编译，最后推送 podspec 到 CocoaPods 的主仓库

				```ruby
					# 打 tag
					git tag 1.0.2 && git push origin --tags
					# podspec 文法检查
					pod spec lint MyFramework.podspec
					# 提交到 CocoaPods 中心仓库
					pod trunk push MyFramework.podspec  
				```
		- Carthage，保证框架 target 能正确编译，然后在 Manage Scheme 里把这个 target 标记为 Shared
		- Swift Package Manager
			- 暂时还不能用于 iOS 项目的依赖管理，但是对于那些并不依赖 iOS 平台的框架来说，现在就可以开始支持 Swift Package Manager 了
			- Swift Package Manager 按照文件夹组织来确定模块，需要把代码放到项目根目录下的 Sources 文件夹里
			- 在根目录下创建 Package.swift 文件来定义 package 信息，在里面定义一个 package 成员，为它指定名字和依赖关系等等。Package Manager 命令将根据这个文件和文件夹的层次来构建你的框架
			
				```
				// Package.swift
				import PackageDescription  
				let package = Package(  
				    name: "MyKit",
				    dependencies: [
				        .Package(url: "https://github.com/onevcat/anotherPacakge.git",
				                 majorVersion: 1)
				    ]
				)
				```
10. 版本管理
	- [Semantic Versioning](http://semver.org/) 和版本兼容 

		> x(major).y(minor).z(patch)

		- major - 公共 API 改动或者删减
		- minor - 新添加了公共 API
		- patch - bug 修正等
		- 0.x.y 只遵守最后一条
	- 最好还是保持 Info.plist 中的 Version 和 Build 和 git tag 的一致性
	- 维护好头文件或 module map 里的版本号信息

11. 持续集成
	- 就 iOS 或者 OSX 开发来说，Travis CI, CircleCI, Coveralls，Codecov 等都是很好的选择
	- [fastlane](https://fastlane.tools/) 是一系列 Cocoa 开发的工具的集合，包括跑测试，打包 app，自动截图，管理 iTunes Connect 等等
	- 建立自己的合适的 Fastfile 文件，然后把想做什么写进去

		```
		# Fastfile
		desc "Release new version"  
		lane :release do |options|  
		  target_version = options[:version]
		  raise "The version is missed." if target_version.nil?
		  ensure_git_branch                                             # 确认 master 分支
		  ensure_git_status_clean                                       # 确认没有未提交的文件
		  scan                                                          # 运行测试

		  sync_build_number_to_git                                      # 将 build 号设为 git commit 数
		  increment_version_number(version_number: target_version)      # 设置版本号

		  version_bump_podspec(path: "Kingfisher.podspec",
		             version_number: target_version)                    # 更新 podspec
		  git_commit_all(message: "Bump version to #{target_version}")  # 提交版本号修改
		  add_git_tag tag: target_version                               # 设置 tag
		  push_to_git_remote                                            # 推送到 git 仓库
		  pod_push                                                      # 提交到 CocoaPods
		end

		$ fastlane release version:1.8.4
		```
		
	- AFNetworking 在 3.0 版本开始加入了 fastlane 做自动集成和发布
12. 创建一个优秀的框架
	- 详尽的文档说明
	- 可以指导后来开发者或者协作者迅速上手的注释
	- 完善的测试
	- 简短易读可维护的代码
	- 让使用者了解版本变化的更新日志
	- issue的解答
	- 不服？跑个分
13. 可能的问题
	- 逻辑上的兼容性
		- 最可能出现问题的地方就是在不同版本中对数据持久化部分的处理是否兼容， 包括数据库和Key-archiving
		- 比如在新版本中添加了一个属性，如何从老版本中进行迁移，如果处理不当，很可能就造成严重错误甚至 crash
	- 重复的依赖
		- 如果对于框架，将 `EMBEDDED_CONTENT_CONTAINS_SWIFT` 设为 `YES` 的话，Swift 运行库将会被复制到框架中，在框架开发中这个 flag 一定是 `NO`，应该在 app 的 target 中进行设置
		- 不要在项目中通过 copy file 把依赖的框架 copy 到框架 target 中，而是应该通过 Podfile 和 Cartfile 来解决依赖问题
	- 依赖可能无法兼容，在框架开发中，如果我们依赖了别的框架，就必须考虑和其他框架及应用的兼容。 为了避免这种依赖无法满足的情况，我们最好尽量选择最宽松的依赖关系
	- 因为现在 Swift 现在 Binary Interface 还没有稳定，不论是框架还是应用项目中所有的 Swift 代码都必须用同样版本的编译器进行编译，就是说，每当 Swift 版本升级，原来 build 过的 framework 需要重新构建否则无法通过编译
	- 有一些开发者表示在转向使用 Framework 以后遇到首次应用加载速度变长的问题，社区讨论和探索结果表明可能是 Dynamic linker 在验证证书的时候的问题。这个时间和 app 中 dynamic framework 的数量为 n^2 时间复杂度。不过现在大家发现这可能是 Apple 在证书管理上的一个 bug，应该是只发生在开发阶段。可能现在比较安全的做法是控制使用的框架数量在合理范围之内。