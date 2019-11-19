###第一种方法：用捏合手势放大缩小
![pinch.gif](http://upload-images.jianshu.io/upload_images/1737720-8442b57871a406ab.gif?imageMogr2/auto-orient/strip)
```
@interface ViewController ()

@property (strong, nonatomic) IBOutlet UIView *redView;
@property (assign, nonatomic) CGFloat scale;//记录上次手势结束的放大倍数
@property (assign, nonatomic) CGFloat realScale;//当前手势应该放大的倍数

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    UIPinchGestureRecognizer *pinchGesture = [[UIPinchGestureRecognizer alloc]initWithTarget:self action:@selector(pinchEvent:)];
    
    [self.view addGestureRecognizer:pinchGesture];
    
    self.scale = 1;
}
- (void)pinchEvent:(UIPinchGestureRecognizer *)pinch {

    self.realScale = self.scale + （pinch.scale - 1）;//当前的放大倍数是上次的放大倍数加上当前手势pinch程度
    
    if (self.realScale > 10) {//设置最大放大倍数
        self.realScale = 10;
    }else if (self.realScale < 0.5){//最小放大倍数
        self.realScale = 0.5;
    }
    
    self.redView.transform = CGAffineTransformMakeScale(self.realScale, self.realScale);
    
    if (pinch.state == UIGestureRecognizerStateEnded){//当结束捏合手势时记录当前图片放大倍数
        
        self.scale = self.realScale;
        
    }

    NSLog(@"%f-------%f",self.scale,self.realScale);
}

@end

```
这种方式有个弊端：如果不进一步设置，放大的焦点只能是从中心开始，而且放大的部分超出屏幕不能滚动查看。
如果想用单击双击手势放大缩小用点击手势UITapGestureRecognizer就可以了，单击设置属性numberOfTapsRequired为1，双击设置为2，就可以了，实现他的点击方法就可以了。

###第二种方法：用scrollView的代理方法实现

![enlarge.gif](http://upload-images.jianshu.io/upload_images/1737720-cc78a2732075c76c.gif?imageMogr2/auto-orient/strip)
设置放大倍数和代理
```
    self.scrollView.minimumZoomScale = 0.5;
    self.scrollView.maximumZoomScale = 10;
    
    self.scrollView.delegate = self;
```
代理方法返回你要放大的图片
```
- (UIView *)viewForZoomingInScrollView:(UIScrollView *)scrollView {
   
    return self.enlargeImage;
}
```
在这个代理方法里面设置滚动范围、调整放大图片的位置（如果不设置，放大后图片按照原来比例frame的X,Y值也会跟随比例变化，图片就跑偏了）
```
- (void)scrollViewDidZoom:(UIScrollView *)scrollView {
    
    CGRect frame = self.enlargeImage.frame;
    
    frame.origin.y = (self.scrollView.frame.size.height - self.enlargeImage.frame.size.height) > 0 ? (self.scrollView.frame.size.height - self.enlargeImage.frame.size.height) * 0.5 : 0;
    frame.origin.x = (self.scrollView.frame.size.width - self.enlargeImage.frame.size.width) > 0 ? (self.scrollView.frame.size.width - self.enlargeImage.frame.size.width) * 0.5 : 0;
    self.enlargeImage.frame = frame;
    
    self.scrollView.contentSize = CGSizeMake(self.enlargeImage.frame.size.width + 30, self.enlargeImage.frame.size.height + 30);
}
```