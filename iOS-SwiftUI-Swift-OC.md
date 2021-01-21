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
#### 取消长按预览
设置属性`allowsLinkPreview`
```
// wkWebView 为 WKWebView 类型
wkWebView.allowsLinkPreview = false
```
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
需要用`inout`关键字
```swift
func increase(_ number: inout Int) {
    number += 1
}
var number = 0
increase(&number)
print("number = \(number)") // 打印：number = 1
```