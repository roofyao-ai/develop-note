# iOS Swift UI & Swift & OC & UIKit常用代码

## 系统功能
### 获取用户当前系统设置语言
```swift
private func currentUserLanguage() -> String {
    guard let languages = UserDefaults.standard.object(forKey: "AppleLanguages") as? NSArray else {
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

## UIKit
### WKWebView
把浏览器内的播放器播放设置为内联播放（默认为false，即全屏播放）
```
let preferences = WKPreferences()		
let configuration = WKWebViewConfiguration()
configuration.preferences = preferences

// 设置为内联播放
configuration.allowsInlineMediaPlayback = true 

let webView = WKWebView(frame: .zero, configuration: configuration)
```