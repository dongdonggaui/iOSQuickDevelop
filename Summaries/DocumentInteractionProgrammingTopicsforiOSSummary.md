# Document Interaction Programming Topics for iOS

[文档地址](https://developer.apple.com/library/ios/documentation/FileManagement/Conceptual/DocumentInteraction_TopicsForIOS)

## Registering the File Types Your App Supports

如何支持

> To declare its support for file types, your app must include the CFBundleDocumentTypes key in its Info.plist property list file. (See Core Foundation Keys.) The system adds this information to a registry that other apps can access through a document interaction controller.

文档类型与文件类型

> A document type usually has a one-to-one correspondence with a particular file type. However, if your app treats more than one file type the same way, you can group those file types together to be treated by your app as a single document type.

字段说明

> Each dictionary in the CFBundleDocumentTypes array can include the following keys:
> 
> * `CFBundleTypeName` specifies the name of the document type.
> * `CFBundleTypeIconFiles` is an array of filenames for the image resources to use as the document’s icon.
> * `LSItemContentTypes` contains an array of strings with the UTI types that represent the supported file types in this group.
> * `LSHandlerRank` describes whether this application owns the document type or is merely able to open it.

## Opening Supported File Types

使用场景

> The system may ask your application to open a specific file and present it to the user. This typically occurs because another application encountered the file and used a document interaction controller to handle it. 

在应用启动时会收到相关信息

> You receive information about the file to be opened in the `application:willFinishLaunchingWithOptions:` or `application:didFinishLaunchingWithOptions:` method of your application delegate.

若要支持相关文件格式，必须实现特定的委托方法

> If your application handles custom file types, you must implement this delegate method (instead of the `applicationDidFinishLaunching:` method) and use it to initialize your application.

需要关注的启动 options 字典中的字段

> * `UIApplicationLaunchOptionsURLKey` contains an NSURL object that specifies the file to open.
> * `UIApplicationLaunchOptionsSourceApplicationKey` contains an NSString with the bundle identifier of the application that initiated the open request.
> * `UIApplicationLaunchOptionsAnnotationKey` contains a property list object that the source application wanted to associate with the file when it was opened.