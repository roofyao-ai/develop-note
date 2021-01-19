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
如何获取`Error`的错误码
```swift
let nsError = error as NSError
print("\(nsError.code)")
```
`kCFURLErrorCannotFindHost`这类错误码常量在swift中如何获得
```swift
let code = CFNetworkErrors.cfurlErrorCannotFindHost.rawValue
print("\(code)") // 打印-1003
```