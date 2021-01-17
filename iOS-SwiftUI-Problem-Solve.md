# iOS Swift UI & Swift & OC & UIKit常见坑以及规避方法

## HStack或VStack的spacing问题
> 使用HStack填充几个需要自定义宽度的View，却发现View的最终组合宽度超出了HStack的范围。

```swift
struct ContentView: View {
    var body: some View {
		GeometryReader { g in
			HStack {
				Color.red
					.frame(width: g.size.width / 2)
				Color.blue
					.frame(width: g.size.width / 2)
			}
		}
    }
}
```
出现这个情况是因为HStack带有spacing属性，且它默认不是0，如果需要严格处理每个子View宽度，正确的写法应该是
```swift
struct ContentView: View {
    var body: some View {
		GeometryReader { g in
			HStack(spacing: 0) {
				Color.red
					.frame(width: g.size.width / 2)
				Color.blue
					.frame(width: g.size.width / 2)
			}
		}
    }
}
```