学习Masonry最好的方式就是下载官方的Demo，简单的布局我就不说了，直接看[https://github.com/SnapKit/Masonry](https://github.com/SnapKit/Masonry)
说说我遇到的坑和需要注意的点：

1.同一个父控件中如果用了Masonry布局，就不能用frame布局了，frame失效，但是可以修改frame.origin.x或者.y的值。Masonry本质上也是frame布局。

2.错误：couldn't find a common superview for。。。，这是用masonry初期很容易犯的错误。错误原因是：没有把要布局的控件添加到父控件，或者添加的父控件的代码在布局代码的后面，或者要布局的控件之间没有相同的父控件。

3.如下图所示的布局，
![Snip20160831_1.png](http://upload-images.jianshu.io/upload_images/1737720-d4ef8a21af5ad2f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

要在一个cell里完成这样的布局：名字与左边头像固定，但名字最长不能覆盖右边按钮，如果这样布局：
```
[self.nameLabel makeConstraints:^(MASConstraintMaker *make) {
        make.left.equalTo(self.headImage.mas_right).offset(9);
        make.centerY.equalTo(self);
        make.right.equalTo(self.addblacklistBtn.mas_left).offset(-2);
    }];
```
不能实现效果。
必须用lessThanOrEqualTo，说明名字与右边按钮的最大距离是多少
```
[self.nameLabel makeConstraints:^(MASConstraintMaker *make) {
        make.left.equalTo(self.headImage.mas_right).offset(9);
        make.centerY.equalTo(self);
        make.right.lessThanOrEqualTo(self.addblacklistBtn.mas_left).offset(-2);
}];
```
或者在用`make.right.equalTo(self.addblacklistBtn.mas_left).offset(-2).priorityLow()`变为低优先级

4 scrollView 上Masonry布局，
为了确定滚动范围 contenSize，需要在 scrollview 上添加一个中间层contenView，再把子控件添加到 contenView 上，其中 contenView 一定要设置 make.edges.equalTo(scrollView);，然后设置子控件的布局。最后再设置 contenView 的宽高，最好是设置 right 和 bottom，最后这一步就是为了确定 scrollview 的contenSize，right要设置为相对最右边的子控件距离， bottom 要设置为相对最下面的子控件的距离
```
         [contenView mas_makeConstraints:^(MASConstraintMaker *make) {
         make.bottom.equalTo(greenView.mas_bottom);
         make.right.equalTo(redView.mas_right);//或者make.width.equalTo(scrollView)
         }];
```
 当然也可以在最右边的子控件中设置 right 和 contenView 的关系，最下面的子控件设置 bottom和 contenView 的关系，这样也可以确定 contenView 的宽高，进而决定了 scrollview 的 contenSize，但是这样代码不方便管理，如果要找如何确定scrollview 的 contenSize，还要去子控件的布局中找。

5.一些小技巧。
在导入masonry时，我们习惯在pch文件中添加`#define MAS_SHORTHAND`，这样就不用写mas_前缀了，但是在用`make.size.mas_equalTo(superview).sizeOffset(CGSizeMake(100, -50))`还要加上mas_。还有就是在确定布局控件关系时，必须用mas_right，mas_left，mas_top，mas_bottom，例如`make.left.equalTo(self.headImage.mas_right).offset(9)`

在offset后面写明距离直接写数字就可以了，但是如果要布局控件与父控件之间的距离，例如要要布局的控件UIImage与他的父控件的左边距离为5，可以简写为`make.left.equaTo(@5)`这时的距离必须要用对象，

如果上下左右的距离相等(5)可以用点语法left.right.top.bottom.equalTo(@5);
内边距用`make.edges.mas_equalTo(UIEdgeInsetsMake(0, 0, 0, 0));`

Autolayout 布局后，不能立刻获取当前的布局控件的frame，最好的layoutSubview方法中获取（controller在viewDidLayoutSubviews方法），也可以在viewDidAppear中获取（太晚了）

6.掌握好masonry的三大方法
(1)布局用`mas_makeConstraints`
(2)重写布局用`mas_remakeConstraints`
(3)更新布局用`mas_updateConstraints`