# iOS Swift UI & Swift & OC & UIKit常见开发坑以及规避方法

## HStack或VStack的spacing问题
> 使用HStack填充几个需要自定义宽度的View，却发现View的最终组合宽度超出了HStack的范围。

```swift
GeometryReader { g in
	HStack {
		Color.red
			.frame(width: g.size.width / 2)
		Color.blue
			.frame(width: g.size.width / 2)
	}
}
```
出现这个情况是因为HStack带有spacing属性，且它默认不是0，如果需要严格处理每个子View宽度，正确的写法应该是
```swift
GeometryReader { g in
	HStack(spacing: 0) {
		Color.red
			.frame(width: g.size.width / 2)
		Color.blue
			.frame(width: g.size.width / 2)
	}
}
```
## http协议地址加载失败
> 在开发的app里访问https成功，访问http却失败了。

苹果限制了对http的访问，如果app想要访问，必须在工程配置文件`Info.plist`中添加如下文本：
```xml
<key>NSAppTransportSecurity</key>
 <dict>
  <key>NSAllowsArbitraryLoads</key>
  <true/>
 </dict>
```
`Info.plist`需要用文本代码编辑 (source code) 的方式打开。
## defer里的代码不执行
> 我把`defer`代码放在函数最后，有时候没有执行？

示例
```swift
func callFunc(completion: @escaping ()->()) {
	if (somethingHappen) {
        return  
    }
    defer {
        completion()
    }
}
```
出现该问题的原因是`defer`机制是运行时处理的，如果在`defer`命令未执行前函数提前`return`了，则`defer`内的代码也不会运行。所以**强烈建议defer放在函数执行的开头。**
正确的代码应该是
```swift
func callFunc(completion: @escaping ()->()) {
    defer {
        completion()
    }
	if (somethingHappen) {
        return  
    }
}
```