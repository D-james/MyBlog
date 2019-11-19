在NavigationController中重写下面方法，来实现隐藏tabBar很常见，但是今天sb了，把 super.pushViewController(viewController, animated: animated)写方法开始了，导致怎么也隐藏不了，我都快跳楼了，浪费了半小时的时间，特此记录下来，以防坑到别的道友。。。
```
override func pushViewController(_ viewController: UIViewController, animated: Bool) {
        
        
        if viewControllers.count > 0 {
         
            viewController.hidesBottomBarWhenPushed = true

        }
        
        super.pushViewController(viewController, animated: animated)
    }
```
super.pushViewController(viewController, animated: animated),这句一定要写最后，如果这句写在方法的开始，他的子界面里再设置viewController.hidesBottomBarWhenPushed = true，会完全不起作用，所以super，一定要写到这个方法最后！！！

另外记录下NavigationBar隐藏的错误姿势
```
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        
        navigationController?.setNavigationBarHidden(true, animated: false)
    }
    
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillAppear(animated)
        
        navigationController?.setNavigationBarHidden(false, animated: true)
    }
```
如果隐藏掉NavigationBar，右划返回就不起作用，需要设置
```
        navigationController?.interactivePopGestureRecognizer?.delegate = self as? UIGestureRecognizerDelegate;
```
这样右划返回手势就起作用了，但是如果设置上面的隐藏NavigationBar的方式会发现当右划返回手势滑到一半没有返回，NavigationBar会出来，尴尬😓

所以正确的方式是
```
 override func viewDidAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        
        navigationController?.setNavigationBarHidden(true, animated: false)
    }
```
最近发现navBar很坑，因为手势问题会偶发界面卡死的问题，还有莫名其妙的隐藏显示失败，所以全部自定义掉最靠谱