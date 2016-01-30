1. 线程与 UI
 	- 建议在application的主线程中接受用户事件和发起界面更新
 	- 这将帮助我们避免在处理用户事件和图形绘制时遇到同步相关的一些麻烦
 	- 在主线程中进行这些操作有利于简化UI的管理逻辑
 	- 在很少一部分特殊情况中，是有利于在其他线程进行图形操作的
	 	- 可以在辅助线程中创建和加工图片，以及一些其他图片相关的操作
	 	- Using secondary threads for these operations can greatly increase performance
	 	- 在辅助线程中进行这些操作会对性能有很大的提升
	 	- 如果你不确定某个图形操作过程能否在辅助线程中进行，那就最好在主线程进行
2. 事件处理的限制
	- 主线程是负责处理事件的
	- 通常情况下，在应用的main函数中会调用UIApplication的run方法，主线程会阻塞在run方法中。（The main thread is the one blocked in the run method）
	- 当Application Kit（UIKit）持续工作时，如果有其他线程卷入了event path，相关操作就会失去秩序
	- 例如，如果有两个不同的线程都对key events（键盘事件？）响应，那么keys的接收就会出现乱序。只让主线程来处理，可以让用户体验更加稳定，一旦主线程接收，如果需要的话主线程可以将事件分发给辅助线程做进一步的处理
	- 可以在辅助线程中调用NSApplication的postEvent:atStart:方法，来将事件发送给主线程的事件队列，但是并不能保证事件的处理序列与用户输入的保持一致。
3. NSView的限制
	- The NSView class is generally not thread-safe. You should create, destroy, resize, move, and perform other operations on NSView objects only from the main thread of an application
	- Drawing from secondary threads is thread-safe as long as you bracket drawing calls with calls to lockFocusIfCanDraw and unlockFocus.
	- If a secondary thread of an application wants to cause portions of the view to be redrawn on the main thread, it must not do so using methods like display, setNeedsDisplay:, setNeedsDisplayInRect:, or setViewsNeedDisplay:. Instead, it should send a message to the main thread or call those methods using the performSelectorOnMainThread:withObject:waitUntilDone: method instead.
	- The view system’s graphics states (gstates) are per-thread
	- Using graphics states used to be a way to achieve better drawing performance over a single-threaded application but that is no longer true. Incorrect use of graphics states can actually lead to drawing code that is less efficient than drawing in the main thread.
4. NSGrapicsContext的限制
	- The NSGraphicsContext class represents the drawing context provided by the underlying graphics system
	- Each NSGraphicsContext instance holds its own independent graphics state: coordinate system, clipping, current font, and so on.
	- An instance of the class is automatically created on the main thread for each NSWindow instance
	- If you do any drawing from a secondary thread, a new instance of NSGraphicsContext is created specifically for that thread.
	- If you do any drawing from a secondary thread, you must flush your drawing calls manually. Cocoa does not automatically update views with content drawn from secondary threads, so you need to call the flushGraphics method of NSGraphicsContext when you finish your drawing. If your application draws content from the main thread only, you do not need to flush your drawing calls
5. NSImage的限制
	- One thread can create an NSImage object, draw to the image buffer, and pass it off to the main thread for drawing.
	- The underlying image cache is shared among all threads