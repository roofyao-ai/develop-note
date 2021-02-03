# iOS Swift UI & Swift & OC & UIKit常用代码和配置

## 系统功能
### 获取用户当前系统设置语言
```swift
private func currentUserLanguage() -> String {
    guard let languages = UserDefaults.standard.object(
                                        forKey: "AppleLanguages") as? NSArray else {
        return ""
    }
    if languages.count == 0 {
        return ""
    }
    let lang = langs[0] as! String
    return lang
}
```
### url域名解析
其实可以不用正则，用URLComponents即可
```swift
if let components = URLComponents(string: "http://www.baidu.com") {
    if let host = components.host {
        print(host)
    }
}
```
### 工程配置http访问权限
苹果限制了对http的访问，如果app想要访问，必须在工程配置文件`Info.plist`中添加如下文本：
```xml
<key>NSAppTransportSecurity</key>
 <dict>
  <key>NSAllowsArbitraryLoads</key>
  <true/>
 </dict>
```
`Info.plist`需要用文本代码编辑 (source code) 的方式打开。
### 多语言
#### 添加语言
`xcode`→`点击项目`→`Info`→`Localizations`→`+号`
#### App名称
新建一个.strings文件：
`新建文件`→`InfoPlist.strings`→`右侧帮助栏`→`Localization`
点击展开文件后，可以看到多个语言的编辑环境，文件内容编辑示例：
```
CFBundleDisplayName = "网易云音乐";
```
其中，CFBundleDisplayName就是要显示的app名的key值，不要写错了。
#### 字符串
`新建文件`→`Localizable.strings`→`右侧帮助栏`→`Localization`
点击展开文件后，可以看到多个语言的编辑环境，文件内容编辑示例：
```
"open in new tab" = "新窗口中打开";
```
编写代码如下
```swift
let newTitle = NSLocalizedString("open in new tab", comment: "")
print("\(newTitle)")
```
以上示例会打印不同语言文件下的`open in new tab`对应的值。
### 线程相关
#### 延迟执行
可以用`DispatchQueue`
```swift
let waitTime: TimeInterval = 10
// 本例中要延迟执行的代码在主线程执行，可以自行更改为后台线程
DispatchQueue.main.asyncAfter(deadline: .now() + waitTime) {
    // 执行想要延迟的代码
}
```
### 检测设备方向
示例代码如下，用一个完整的类封装
```swift
import Foundation
import UIKit

class DeviceOrientation {
	static let instance = DeviceOrientation()
	private weak var observer: NSObjectProtocol!
	private var _orientation: UIDeviceOrientation = .unknown
	var orientation: UIDeviceOrientation {
		return _orientation
	}
	
	private init() {
		observer = NotificationCenter.default.addObserver(
                    forName: UIDevice.orientationDidChangeNotification,
			        object: nil, queue: nil ) { [weak self] notification in
			guard let self = self else {
				return
			}
			
			guard let device = notification.object as? UIDevice else {
				return
			}
			
			if device.orientation.isPortrait || device.orientation.isLandscape {
				self._orientation = device.orientation
			}
		}
	}
	
	deinit {
		if observer != nil {
			NotificationCenter.default.removeObserver(observer!)
		}
	}
}
```
## SwiftUI
### TextField
#### 去掉键盘自动纠错
添加修饰符
```swift
.disableAutocorrection(true)
```
#### 去掉智能首字母大写纠正
添加修饰符
```swift
.autocapitalization(.none)
```

### 视觉适配
#### 忽略边缘安全区域
```swift
.edgesIgnoringSafeArea(.all) // .bottom, .top ...
```
### 响应`UIApplicationDelegate`回调
完整代码示例如下
```swift
import SwiftUI

@main
struct App: App {
  @UIApplicationDelegateAdaptor(AppDelegate.self) var appDelegate
}

class AppDelegate: NSObject {
}

extension AppDelegate: UIApplicationDelegate {
  func application(_ application: UIApplication, 
    supportedInterfaceOrientationsFor window: UIWindow?) -> UIInterfaceOrientationMask {
      return [.portrait, .landscapeLeft]
  }
}
```

## UIKit
### WKWebView
#### 把浏览器内的播放器播放设置为内联播放（默认为false，即全屏播放）
```swift
let preferences = WKPreferences()		
let configuration = WKWebViewConfiguration()
configuration.preferences = preferences

// 设置为内联播放
configuration.allowsInlineMediaPlayback = true 

let webView = WKWebView(frame: .zero, 
                        configuration: configuration)
```
#### 加载本地html文本
```swift
let content = "<html>content</html>" // 指定文本
let url = Bundle.main.bundleURL
webView.loadHTMLString(content, baseURL: url)
```
#### 取消长按链接预览
设置属性`allowsLinkPreview`
```swift
// wkWebView 为 WKWebView 类型
wkWebView.allowsLinkPreview = false
```
#### 自适配长按链接预览及弹出菜单
首先要确保`allowsLinkPreview`的属性为`true`。
```swift
webView.allowsLinkPreview = true
```
配置`WKWebView`的回调协议`uiDelegate`，该协议名为`WKUIDelegate`。
```swift
webView.uiDelegate = self
```
接收`WKUIDelegate`的以下回调
```swift
func webView(_ webView: WKWebView, 
            contextMenuConfigurationForElement elementInfo: WKContextMenuElementInfo, 
            completionHandler: @escaping (UIContextMenuConfiguration?) -> Void)
```
在回调中执行代码示例如下
```swift
completionHandler(
    UIContextMenuConfiguration(identifier: nil, 
                               previewProvider: { () -> UIViewController? in
        if let url = elementInfo.linkURL {
            return SFSafariViewController(url: url) // 需要 import SafariServices
        } else {
            return nil
        }
    }, actionProvider: { (_) -> UIMenu? in
        let newTitle = NSLocalizedString("open in new tab", comment: "")
        let newCommand = UIAction(title: newTitle, image: nil, state: .off) { (_) in
            print("new")
        }
        let oldCommand = UIAction(title: "old", image: nil, state: .off) { (_) in
            print("old")
        }
        return UIMenu(title: "", options: .destructive, children: [newCommand, oldCommand])
    }))
```
解释一下，该回调通过开发者调用`completionHandler`传入**预览**和**菜单选项**的配置。
其中`previewProvider`是预览部分，这部分依然需要你提供一个代码块，在代码块里返回一个预览Controller，可以用苹果提供的`SFSafariViewController`。如果要用到`SFSafariViewController`，那么需要在文件头加入
```swift
import SafariServices
```
因为是链接预览，参数`elementInfo`提供了`url`值可以用于传入。
而`actionProvider`是菜单栏部分，这部分也需要你提供一个代码块，在代码里返回所需菜单栏。
更多相关信息可以查阅苹果文档。
### 用户点击链接跳转的事件捕捉
通过`WKWebView`的`navigationDelegate`回调来捕捉，协议类型为`WKNavigationDelegate`。
回调函数
```swift
func webView(
        _ webView: WKWebView,
        decidePolicyFor navigationAction: WKNavigationAction,
        preferences: WKWebpagePreferences,
        decisionHandler: @escaping (WKNavigationActionPolicy, WKWebpagePreferences) -> Void)
```
通过该函数的参数`navigationAction`类型为`WKNavigationAction`，其属性`navigationType`类型为`WKNavigationType`，该类型为枚举类型
```swift
public enum WKNavigationType : Int {
    case linkActivated = 0 // 点击链接时的navigationType将会是这个值
    case formSubmitted = 1
    case backForward = 2
    case reload = 3
    case formResubmitted = 4
    case other = -1
}
```
因此，在回调函数中实现如下即可捕获点击事件
```swift
if navigationAction.navigationType == .linkActivated {
    print("有链接被点击了。")
}
```
#### 获取快照图片
函数`takeSnapshot`
```swift
wkWebView.takeSnapshot(with: WKSnapshotConfiguration()) { (image, error) in
    if let error = error {
        print("发生错误: \(error)")
    }
    if let image = image {
        print("获得图片 = \(image)")
    }
}
```
### UIScrollView
#### 如何获得点击状态栏事件的回调
点击状态栏事件后，`UIScrollView`的内容会自动滚动到顶部内容。
要想激活或者关闭该功能，需要配置布尔属性`isScrollEnabled`，该属性默认为`true`。
```swift
scrollView.isScrollEnabled = true
```
要获得点击事件的回调，需要配置`scrollView`的`delegate`。
```swift
scrollView.delegate = self
```
其中，接受回调的对象必须支持协议`UIScrollViewDelegate`。
每次用户点击状态栏，`UIScrollView`会回调 `scrollViewWillBeginDragging`
```swift
func scrollViewShouldScrollToTop(_ scrollView: UIScrollView) -> Bool {
    return true
}
```
该回调会在用户每次点击顶部状态栏时发生。
## 基础类型
### String
#### 替换字符串
用`replacingOccurrences`函数。
```swift
let title = "hello, world."
let newTitle = title.replacingOccurrences(
                        of: ".",
                        with: "!")
print(newTitle) // 输出结果： hello, world!
```
#### url编解码
函数`addingPercentEncoding`和`removingPercentEncoding`
```swift
let text = "小明"
if let encodedData = text.addingPercentEncoding(withAllowedCharacters: .urlHostAllowed) {
    print("\(encodedData)") // 打印：%E5%B0%8F%E6%98%8E
    if let decodedData = encodedData.removingPercentEncoding {
        print("decodedData: \(decodedData)") // 打印：小明
    }
}
```
### Error
#### 如何获取回调函数中`Error`的错误码
注意，前提条件是`error is NSError`
```swift
let nsError = error as NSError
print("\(nsError.code)")
```
`kCFURLErrorCannotFindHost`这类错误码常量在swift中如何获得
```swift
let code = CFNetworkErrors.cfurlErrorCannotFindHost.rawValue
print("\(code)") // 打印-1003
```
### struct
#### 把struct引用传递给函数参数
struct属于值类型，需要用`inout`关键字，以下例子用Int类型，struct同理。
```swift
func increase(_ number: inout Int) {
    number += 1
}
var number = 0
increase(&number)
print("number = \(number)") // 打印：number = 1
```
### Image
#### 把`UIKit`的`UIImage`转为`Image`
Image构造函数自带
```swift
let image = Image(uiImage: uiImage)
```
## Swift语言特性
### defer用法
`defer`非常有用，除了在代码里在函数结束前要执行清理、关闭操作之外，还有另外一个重要用途，就是用来执行函数回调的completion handler。
```swift
func callFunc(completion: @escaping ()->()) {
    defer {
        completion()
    }
    return
}
```
**强烈建议defer放在函数执行的开头。**
因为`defer`是一个运行时机制，也就是说，当你在还未运行`defer`代码时`return`了，`defer`将不会运行。
```swift
// 错误代码示例
func callFunc(completion: @escaping ()->()) {
    if (somethingHappen) {
        // 一旦进入这个分支，则completion将不会运行！
        return  
    }
    defer {
        completion()
    }
}
```

### 各种常量写法
#### 多进制
代码示例
```swift
let i = 10 // 十进制
let x = 0x1f // 十六进制
let o = 0o27 // 八进制 
let b = 0b0010 // 二进制
```
#### 数字的辅助线
代码示例
```swift
let oneMillion = 1_000_000
```
