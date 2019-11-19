![Snip20190712_3.png](https://upload-images.jianshu.io/upload_images/1737720-127db0bafd939532.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

思路：
	1.因为tabBar有一个超出tabBar的凸起效果，所以需要自定义tabBar
	2.需要重新布局tabBarBtn，给中间按钮腾出位置
	3.tabBar的背景图需要拉伸，处理中间按钮超出位置不响应的问题
	
###1.自定义tabBar	
UITabbarController的tabBar 是一个只读属性 不能够直接赋值, 需要使用KVC赋值
`[self setValue:[MyTabBar new] forKey:@"tabBar"];`
1.给tabbar添加背景图片
1>为保证自定义背景凸起图片能超出tabbar，不能直接设置tabbar的背景图片，需要在tabbar上添加subview背景图片，才能设置frame超出tabbar

2>保证背景凸起图片拉伸，不变形，
```
UIImage *bgImage = [image resizableImageWithCapInsets:UIEdgeInsetsMake(20, 0, 0, 0) resizingMode:UIImageResizingModeStretch];
```
这种方式还是会变形，因为图片周围不规则，还要中间凸起居中，所以图片宽最好是320pt，这样变形不明显，不然这个只能图片拼接，或者自己画一个了（麻烦）。

3>去掉tabbar自带黑线
```
    [self setBackgroundImage:[UIImage new]];
    [self setShadowImage:[UIImage new]];
```
###2.自定义中间凸起按钮
1>在`layoutSubviews`中重新布局tabbarItem，计算中间按钮的位置
```
	// layoutSubviews遍历子控件寻找UITabBarButton，给UITabBarButton重新设置frame
- (void)layoutSubviews {
    [super layoutSubviews];
    
    CGFloat width = 50;
    self.centerTabBarBtn.frame = CGRectMake((ScreenWidth-width)/2, -5, width, width);
    
    CGFloat tabBarButtonW = ScreenWidth / 5;
    CGFloat tabBarButtonIndex = 0;
    for (UIView *child in self.subviews) {

        Class class = NSClassFromString(@"UITabBarButton");
        if ([child isKindOfClass:class]) {

            // 重新设置frame
            CGRect frame = CGRectMake(tabBarButtonIndex * tabBarButtonW, 0, tabBarButtonW, 49);
            child.frame = frame;

            // 增加索引
            if (tabBarButtonIndex == 1) {
                tabBarButtonIndex++;
            }
            tabBarButtonIndex++;
        }
    }
}
```
2>第二种方式可以`TabBarController`多`[self addChildViewController:nav];`一次空的controller，然后在UITabBarControllerDelegate方法中阻止点击事件
```

- (BOOL)tabBarController:(UITabBarController *)tabBarController shouldSelectViewController:(UIViewController *)viewController
{
    if ([viewController isKindOfClass:[RubbishViewController class]]){
            return NO;
 	}
    return YES;
}
```
我目前就是这样做的，为啥呢？我的第一种方式layoutSubviews遍历子控件UITabBarButton的顺序是乱的
	
###3.其他问题处理	
1>防止点击无效，重写hitTest方法
```
//处理超出区域点击无效的问题
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event{
    if (self.hidden){
        return [super hitTest:point withEvent:event];
    }else {
        //转换坐标
        CGPoint tempPoint = [self.centerTabBarBtn convertPoint:point fromView:self];
        //判断点击的点是否在按钮区域内
        if (CGRectContainsPoint(self.centerTabBarBtn.bounds, tempPoint)){
            //返回按钮
            return self.centerTabBarBtn;
        }else {
            return [super hitTest:point withEvent:event];
        }
    }
}

```
代码地址：[https://github.com/D-james/CustomTabbarViewController](https://github.com/D-james/CustomTabbarViewController)