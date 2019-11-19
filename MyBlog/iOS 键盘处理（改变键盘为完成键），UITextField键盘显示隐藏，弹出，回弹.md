#####很多时候用到UITextField时，处理键盘是一个很棘手的问题。

###问题一：如何隐藏键盘？
####方案1.改变键盘右下角的换行（enter）键为完成键，后实现代理方法键盘自动回弹

![keyBoardControll.gif](http://upload-images.jianshu.io/upload_images/1737720-9dd89a4f26eda333.gif?imageMogr2/auto-orient/strip)
```
UITextField *textField = [[UITextField alloc]initWithFrame:CGRectMake(100, 300, 200, 40)];
[self.view addSubview:textField];
    
textField.delegate = self;
textField.returnKeyType = UIReturnKeyDone;//改变为完成键，如果在项目中导入了YYText框架那么原生的就被替换掉了，变为returnKeyType = UIKeyboardTypeTwitter;

//实现UITextField代理方法
- (BOOL)textFieldShouldReturn:(UITextField *)textField {
    
    [textField resignFirstResponder];//取消第一响应者
    
    return YES;
}
/*textField.returnKeyType可以改变为很多样式
typedef NS_ENUM(NSInteger, UIReturnKeyType) {
    UIReturnKeyDefault,
    UIReturnKeyGo,
    UIReturnKeyGoogle,
    UIReturnKeyJoin,
    UIReturnKeyNext,
    UIReturnKeyRoute,
    UIReturnKeySearch,
    UIReturnKeySend,
    UIReturnKeyYahoo,
    UIReturnKeyDone,
    UIReturnKeyEmergencyCall,
    UIReturnKeyContinue NS_ENUM_AVAILABLE_IOS(9_0),
};
*/
```
当然搜狗输入法是自带隐藏键盘的功能的，但是你不能保证每个用户都装有搜狗输入法，这种方案也会改变搜狗键盘的右下角按钮为完成键
####方案2.点击textField以外区域键盘回弹
```
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    [self.view endEditing:YES];
}
```
 如果textField在tableView上还可以实现下面的tableView的代理方法
```
-(void)tableView:(UITableView*)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath{
    [self.view endEditing:YES];
}
```

###问题二：键盘键盘遮挡输入框的的问题
解决方案
```
#pragma mark - textFieldDelegate（别忘了遵守协议设置代理）
- (void)textFieldDidBeginEditing:(UITextField *)textField {
   self.view.y = self.view.y - 216;  //216是输入框在最底部时view移动的距离，具体移动多少距离，需要根据实际情况而定
}

- (void)textFieldDidEndEditing:(UITextField *)textField {
  self.view.y = self.view.y + 216;
}
```

更好的方案，直接监听键盘通知
```
// 键盘出现的通知
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(keyboardWasShown:) name:UIKeyboardDidShowNotification object:nil];
- (void) keyboardWasShown:(NSNotification *)notification {
CGRect frame = [[[x userInfo] objectForKey:UIKeyboardFrameEndUserInfoKey] CGRectValue];
        CGFloat height = frame.size.height;
        if (height > 300) {
            [self.inputView mas_updateConstraints:^(MASConstraintMaker *make) {
                make.bottom.mas_equalTo(-height);
            }];
            [UIView animateWithDuration:0.1 animations:^{
                [self.view layoutIfNeeded];
            }];
        }
}
    // 键盘消失的通知
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(keyboardWillBeHiden:) name:UIKeyboardWillHideNotification object:nil];

- (void) keyboardWillBeHiden:(NSNotification *)notification { 
        [self.inputView mas_updateConstraints:^(MASConstraintMaker *make) {
            make.bottom.mas_equalTo(-BottomSpace);
        }];
        [UIView animateWithDuration:0.1 animations:^{
            [self.view layoutIfNeeded];
        }];
}
```

当然还可以用[IQKeyboardManager](https://github.com/hackiftekhar/IQKeyboardManager)比较成熟的框架，但是用这个框架输入框最好在scrollview上，或者继承他的tableview上，否则会出现导航栏上移的问题。
当输入框不加在scrollview上，最好还是自定义，自己写键盘通知的方法