###第一种：系统自带压缩方式
```
NSData * __nullable UIImageJPEGRepresentation(UIImage * __nonnull image, CGFloat compressionQuality)
```
用法：NSData *dataImage =  UIImageJPEGRepresentation(image, 0.1);
当然也有很多人用这种方式image转data

优点：在基本没有降低图片的质量的前提下，压缩图片，不改变图片的分辨率
缺点：压缩图片有一定限度，因为这是不改变分辨率的压缩。比如你想压缩图片到原来的十分之一大小，但是他最大可能只会压缩到三分之一。经测试图片最大能压到的大小和图片本身有关，每个图片各不相同

###第二种：通过改变图片尺寸压缩图片
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
这个方法我写成一个UIImage的分类，方便使用

优点：可以缩小图片到任意大小，可以自定义压缩后图片的尺寸
缺点：改变图片的分辨率，会大大降低图片的质量