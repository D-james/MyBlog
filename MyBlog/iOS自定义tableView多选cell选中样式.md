前段时间写了一篇tableView多选删除的文章[http://www.jianshu.com/p/abfc4d7e56a9](http://www.jianshu.com/p/abfc4d7e56a9)，然后一直有人问怎么自定义多选状态的按钮样式，系统默认是蓝色的：就像这样
![系统自带效果](http://upload-images.jianshu.io/upload_images/1737720-9984f30951895136.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们想实现右边的蓝色按钮自定义样式，我找了个橙色的图片（当然实现什么效果看你找什么样的图片），实现的效果是这样的，
![实现自定义的效果](http://upload-images.jianshu.io/upload_images/1737720-704f9c7c4d2b47ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##注意：如果只是想改变左边选中颜色，只要改变cell前景色就可以cell.tintColor = [UIColor orangeColor];
如果要替换选中的图片在往下看，我的思路是这样的：首先看这个选中状态按钮的层级关系
![按钮的层级关系图1](http://upload-images.jianshu.io/upload_images/1737720-e818f1f71a98763c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![按钮的层级关系图2](http://upload-images.jianshu.io/upload_images/1737720-17ec6fa60163baa0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
看到了层级关系，这个选中状态按钮不是一个button而是一个UIImageView，这个UIImageView在UITableViewCellEditControl上面，这个类没见过，但是看见了Control的后缀他一定继承自UIControl，确定了他的层级关系，第一个想法是在初始化cell的方法中把选中图片替换掉就可以了
```
cell = [[UITableViewCell alloc]initWithStyle:UITableViewCellStyleDefault reuseIdentifier:cellID];
        for (id subCell in cell.subviews) {
            if ([subCell isKindOfClass:[UIControl class]]) {
                
                for (UIImageView *circleImage in [subCell subviews]) {
                        circleImage.image = [UIImage imageNamed:@"CellButtonSelected"];
                    
                }
            }
        }
```
但是试了之后发现不管用,不进去最后设置image的方法，想想这个选中图片有两种状态选中和未选中状态，我要设置的是选中状态的图片，而cell初始化没有imageView，所以在在自定cell布局中写，或者在didSelectRowAtIndexPath的方法中写,先说在didSelectRowAtIndexPath的方法
```
 NSArray *subviews = [[tableView cellForRowAtIndexPath:indexPath] subviews];
    for (id subCell in subviews) {
        if ([subCell isKindOfClass:[UIControl class]]) {
            
            for (UIImageView *circleImage in [subCell subviews]) {
                circleImage.image = [UIImage imageNamed:@"CellButtonSelected"];
            }
        }
        
    }
```
点击cell，管用！这种方式可以达到不想要cell的选中样式的效果，平时你设置cell.selectionStyle = UITableViewCellSelectionStyleNone;这样cell左边的选中圆圈也不见了，但是如果在didDeselectRowAtIndexPath中设置未选中图片圆圈，在didSelectRowAtIndexPath中设置选中图片，左边选中圆圈就显示了。
```
- (void)tableView:(UITableView *)tableView didDeselectRowAtIndexPath:(NSIndexPath *)indexPath {
    UITableViewCell *cell = [tableView cellForRowAtIndexPath:indexPath];
    NSArray *subviews = cell.subviews;
    for (id subCell in subviews) {
        if ([subCell isKindOfClass:[UIControl class]]) {
            
            for (UIImageView *circleImage in [subCell subviews]) {

                circleImage.image = [UIImage imageNamed:@"CellButton"];

            }
        }
        
    }
```
效果是这样的，没有了cell的选中样式

![没有了cell的浅蓝色选中样式](http://upload-images.jianshu.io/upload_images/1737720-37777523ab69557e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
但是有一个问题就是全选的时候还是系统自带的蓝色圆圈，因为全选的方法是
```
for (int i = 0; i < self.dataArr.count; i++) {
            NSIndexPath *indexPath = [NSIndexPath indexPathForRow:i inSection:0];
            [self.tableView selectRowAtIndexPath:indexPath animated:NO scrollPosition:UITableViewScrollPositionBottom];
}
```
没有调用didSelectRowAtIndexPath方法，所以在全选点击后要替换每个蓝色圆圈
```
for (int i = 0; i < self.dataArr.count; i++) {
            NSIndexPath *indexPath = [NSIndexPath indexPathForRow:i inSection:0];
            [self.tableView selectRowAtIndexPath:indexPath animated:NO scrollPosition:UITableViewScrollPositionBottom];
            NSArray *subviews = [[self.tableView cellForRowAtIndexPath:indexPath] subviews];
            for (id subCell in subviews) {
                if ([subCell isKindOfClass:[UIControl class]]) {
                    
                    for (UIImageView *circleImage in [subCell subviews]) {
                        circleImage.image = [UIImage imageNamed:@"CellButtonSelected"];
                    }
                }
                
            }   
        }
```
还有一种简单的方法在自定义的cell里面布局里面设置，就不用再在全选点击事件中加入上面的代码了：
```
-(void)layoutSubviews
{
   
    for (UIControl *control in self.subviews){
        if ([control isMemberOfClass:NSClassFromString(@"UITableViewCellEditControl")]){
            for (UIView *view in control.subviews)
            {
                if ([view isKindOfClass: [UIImageView class]]) {
                    UIImageView *image=(UIImageView *)view;
                    if (self.selected) {
                        image.image=[UIImage imageNamed:@"CellButtonSelected"];
                    }
                    else
                    {
                        image.image=[UIImage imageNamed:@"CellButton"];
                    }
                }
            }
        }
    }
    
    [super layoutSubviews];
}
```
但是这两种方法都有一个问题，就是当你长按一个按钮时看到的还是系统的蓝色圆圈，还要改变长按编辑的代码
```
- (void)setEditing:(BOOL)editing animated:(BOOL)animated
{
    [super setEditing:editing animated:animated];
        for (UIControl *control in self.subviews){
            if ([control isMemberOfClass:NSClassFromString(@"UITableViewCellEditControl")]){
                for (UIView *view in control.subviews)
                {
                    if ([view isKindOfClass: [UIImageView class]]) {
                        UIImageView *image=(UIImageView *)view;
                        if (!self.selected) {
                            image.image=[UIImage imageNamed:@"CellButton"];
                        }
                    }
                }
            }
        }
    
}
```
GitHub地址：[https://github.com/D-james/MultipleSelectCell](https://github.com/D-james/MultipleSelectCell)
希望可以帮到你