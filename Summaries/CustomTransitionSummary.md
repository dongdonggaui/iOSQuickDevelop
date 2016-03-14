1. 创建自定义动画类，实现`UIViewControllerAnimatedTransitioning`协议，实现`transitionDuration:`和`animateTransition:`方法，在`animateTransition:`方法中通过`transitionContext`参数获取动画所需要的上下文信息，如`containView`、`fromVC`、`toVC`等，然后进行自定义动画的实现
2. 实现`transition`需要的协议
	- 若为`UINavigationController`的`push`和`pop`，创建一个类实现`UINavigationControllerDelegate`协议的相关方法
		- 实现`navigationController:interactionControllerForAnimationController:`获取可交互的transition动画控制器，若不需要可交互返回`nil`即可
		- 实现`navigationController:didShowViewController:animated:`做一些必要的额外工作，如更新 statusBar 样式等
		- 若支持交互，建议在此类中实现`UIScreenEdgePanGestureRecognizer`手势回调
			- 在识别手势`.Began`之后将`transition`标识为`interacting`
			- 为`transition`创建`UIPercentDrivenInteractiveTransition`对象，并`startInteractiveTransition`
			- 在`.Change`中`updateInteractiveTransition`
			- 手势结束时（`.Ended`和`.Cancel`）中根据`percent`判断是`finishInteractiveTransition`还是`cancelInteractiveTransition`
			- 一些收尾工作，如回收`UIPercentDrivenInteractiveTransition`对象
		- 将`UINavigationController`的`delegate`设置为此类的实例
	- 若为`PresentTransition`，创建一个类实现`UIViewControllerTransitioningDelegate`协议的相关方法
		- 实现`animationControllerForPresentedController:presentingController:sourceController:`获取 present 动画对象
		- 实现 `animationControllerForDismissedController:` 获取 dismiss 动画对象
		- 实现 `interactionControllerForDismissal:` 获取可交互的 dismiss 动画对象
		- 实现 `interactionControllerForPresentation:` 获取可交互的  present  动画对象
		- 将想要 present / dismiss 的 viewController 的 `transitioningDelegate` 设置为此类的实例