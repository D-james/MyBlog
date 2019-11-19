当在tabbarVC上nav的rootVC上添加WKWebview时，WKWebview会显示不全,解决方式：
```
self.edgesForExtendedLayout = .bottom//OC设置为UIRectEdgeNone
let webView = WKWebView.init(frame: CGRect(x: 0, y: 0, width: ScreenWidth, height: ScreenHeight - 113))
automaticallyAdjustsScrollViewInsets = false
```
我这个减去的113是， 64(navbar) + 49(tabbar) 
只要WKWebview添加到nav上就会显示不全。需要设置edgesForExtendedLayout ，这是UIViewController的属性
还要设置automaticallyAdjustsScrollViewInsets = false,否则底部上拉和顶部下拉，webView的frame 会变化