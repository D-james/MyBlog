##本次更新三个常用的方法，全部写成了 UIImage 的分类方法

##截屏
```
//截屏
+ (UIImage *)snapScreen {
    
    UIWindow *window = [UIApplication sharedApplication].keyWindow;
    
    UIGraphicsBeginImageContextWithOptions(window.bounds.size, false, [UIScreen mainScreen].scale);
    [window drawViewHierarchyInRect:window.bounds afterScreenUpdates:false];
    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    
    return image;
}
```
##压缩图片
```
//压缩图片
+ (UIImage*)imageWithImage:(UIImage*)image scaledToSize:(CGSize)newSize
{
    // Create a graphics image context
    UIGraphicsBeginImageContext(newSize);
    
    // Tell the old image to draw in this new context, with the desired
    // new size
    [image drawInRect:CGRectMake(0,0,newSize.width,newSize.height)];
    
    // Get the new image from the context
    UIImage* newImage = UIGraphicsGetImageFromCurrentImageContext();
    
    // End the context
    UIGraphicsEndImageContext();
    
    // Return the new image.
    return newImage;
}
```
##拉伸图片
```
//拉伸图片
+ (UIImage *)resizableImageName:(NSString *)imageName {
    
    UIImage *oldBackgroundImage = [Utility imageNamedWithFileName:imageName];
    CGFloat top = oldBackgroundImage.size.height * 0.5;
    CGFloat left = oldBackgroundImage.size.width * 0.5;
    CGFloat bottom = oldBackgroundImage.size.height * 0.5;
    CGFloat right = oldBackgroundImage.size.width * 0.5;
    
    UIEdgeInsets edgeInsets = UIEdgeInsetsMake(top, left, bottom, right);
    UIImageResizingMode mode = UIImageResizingModeStretch;
    UIImage *newBackgroundImage = [oldBackgroundImage resizableImageWithCapInsets:edgeInsets resizingMode:mode];
    
    return newBackgroundImage;
}
```
