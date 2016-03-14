在阅读 [Alamofire](https://github.com/Alamofire/Alamofire) 源码时，看到一段关于 User-Agent 的代码：

~~~swift
let userAgent: String = {
    if let info = NSBundle.mainBundle().infoDictionary {
        let executable: AnyObject = info[kCFBundleExecutableKey as String] ?? "Unknown"
        let bundle: AnyObject = info[kCFBundleIdentifierKey as String] ?? "Unknown"
        let version: AnyObject = info[kCFBundleVersionKey as String] ?? "Unknown"
        let os: AnyObject = NSProcessInfo.processInfo().operatingSystemVersionString ?? "Unknown"

        var mutableUserAgent = NSMutableString(string: "\(executable)/\(bundle) (\(version); OS \(os))") as CFMutableString
        let transform = NSString(string: "Any-Latin; Latin-ASCII; [:^ASCII:] Remove") as CFString

        if CFStringTransform(mutableUserAgent, UnsafeMutablePointer<CFRange>(nil), transform, false) {
            return mutableUserAgent as String
        }
    }

    return "Alamofire"
}()

~~~