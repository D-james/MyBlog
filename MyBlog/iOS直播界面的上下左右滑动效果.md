我们看现在主流的直播平台都有左右滑动去掉直播信息，上下滑动切换频道的功能:
粉色界面代表直播信息界面，就是观众、礼物、弹幕的界面。
下面的图片代表直播的播放界面，用切换图片代表频道的切换。
![slidLivingPage.gif](http://upload-images.jianshu.io/upload_images/1737720-60da15d795d2c42e.gif?imageMogr2/auto-orient/strip)
在这我写了一个小的Demo来说明是怎么实现这个功能的：首先说一下思路
1. 看到这个界面的第一反应这不就是一个ScrollView吗？但仔细把玩一下，ScrollView不好实现这个效果。必须用获取手指触摸位置拖动控件的方法，在有直播信息的界面，移除直播信息界面划过屏幕宽的大于五分之一才回移除直播信息界面。在没有直播信息界面时，必须划过小于屏幕宽的五分之四才能添加直播信息界面。上下切花频道，大于屏幕高的五分之一会切换到下一频道，小于负的屏幕高五分之一就会切换到上移频道。切换频道需要重新加载界面。上述滑动条件如果没有满足都会回到滑动前的位置。
2. 首先单独实现左右滑动，再单独实现上下滑动
3. 避免左右滑和上下滑冲突，区别开始滑动时，是上下滑动和还是左右滑动，开始滑动后水平或垂直滑动的方向不能改变，直到再次开始触摸
4. 在结束触摸的方法中判断，实现上下滑动切换频道播放，左右滑动添加和移除直播信息界面

在控制器上加上直播信息LivingInfoView，下面代码中的注释已经说的很清楚了
手势分两部分
####播放控制器的代码：
 ```
#define Screen_width [UIScreen mainScreen].bounds.size.width
#define Screen_height [UIScreen mainScreen].bounds.size.height

@interface ViewController ()

@property (weak, nonatomic) LivingInfoView *livingInfoView;
@property (assign, nonatomic) BOOL isBeginSlid;//记录开始触摸的方向
@property (weak, nonatomic) UIImageView *playLayer;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    self.view.backgroundColor = [UIColor whiteColor];
    
//    播放层
    UIImageView *playLayer = [[UIImageView alloc]initWithFrame:CGRectMake(0, 0, Screen_width, Screen_height)];
    [self.view addSubview:playLayer];
    self.playLayer = playLayer;
    
    playLayer.image = [UIImage imageNamed:@"PlayView"];
    playLayer.userInteractionEnabled = YES;
//    播放信息层
    LivingInfoView *livingInfoView = [[LivingInfoView alloc]initWithFrame:CGRectMake(0, 0, Screen_width, Screen_height)];
    [playLayer addSubview:livingInfoView];
    self.livingInfoView = livingInfoView;
    
    livingInfoView.backgroundColor = [UIColor colorWithRed:242 / 255.0 green:156 / 255.0 blue:177 / 255.0 alpha:0.6];
}

#pragma mark - 界面的滑动
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    self.isBeginSlid = YES;
}
```
开始记录触摸方向，touchMove的方法是滑动中记录手指移动的位置，是界面滑动的核心方法，直播信息界面的touchMove方法会传到下层的直播播放界面，所以播放页的touchMove方法可以和直播信息界面的touchMove方法合并。
```
- (void)touchesMoved:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {

    //    1.获取手指
    UITouch *touch = [touches anyObject];
    //    2.获取触摸的上一个位置
    CGPoint lastPoint;
    CGPoint currentPoint;
    
    //    3.获取偏移位置
    CGPoint tempCenter;
    
    if (self.isBeginSlid) {//首次触摸进入
        lastPoint = [touch previousLocationInView:self.playLayer];
        currentPoint = [touch locationInView:self.playLayer];
        
        
        //判断是左右滑动还是上下滑动
        if (ABS(currentPoint.x - lastPoint.x) > ABS(currentPoint.y - lastPoint.y)) {
            //    3.获取偏移位置
            tempCenter = self.livingInfoView.center;
            tempCenter.x += currentPoint.x - lastPoint.x;//左右滑动
            //禁止向左划
            if (self.livingInfoView.frame.origin.x == 0 && currentPoint.x -lastPoint.x > 0) {//滑动开始是从0点开始的，并且是向右滑动
                self.livingInfoView.center = tempCenter;
                
            }
//            else if(self.livingInfoView.frame.origin.x > 0){
//                self.livingInfoView.center = tempCenter;
//            }
//            NSLog(@"%@-----%@",NSStringFromCGPoint(tempCenter),NSStringFromCGPoint(self.livingInfoView.center));
        }else{
            //    3.获取偏移位置
            tempCenter = self.playLayer.center;
            tempCenter.y += currentPoint.y - lastPoint.y;//上下滑动
            self.playLayer.center = tempCenter;
        }
    }else{//滑动开始后进入，滑动方向要么水平要么垂直
        if (self.playLayer.frame.origin.y != 0){//垂直的优先级高于左右滑，因为左右滑的判定是不等于0
            
            lastPoint = [touch previousLocationInView:self.playLayer];
            currentPoint = [touch locationInView:self.playLayer];
            tempCenter = self.playLayer.center;
            
            tempCenter.y += currentPoint.y -lastPoint.y;
            self.playLayer.center = tempCenter;
        }else if (self.livingInfoView.frame.origin.x != 0) {
            
            lastPoint = [touch previousLocationInView:self.livingInfoView];
            currentPoint = [touch locationInView:self.livingInfoView];
            tempCenter = self.livingInfoView.center;
            
            tempCenter.x += currentPoint.x -lastPoint.x;
            
            //禁止向左划

            if (self.livingInfoView.frame.origin.x == 0 && currentPoint.x -lastPoint.x > 0) {//滑动开始是从0点开始的，并且是向右滑动
                self.livingInfoView.center = tempCenter;
        
            }else if(self.livingInfoView.frame.origin.x > 0){
                self.livingInfoView.center = tempCenter;
            }
            
        }
    }
  
    self.isBeginSlid = NO;
}
```
###滑动完成后要判断滑动结束时的位置
####LivingInfoView上的手势：
touchEnd和touchCancell的代码是一样的，判定滑动结束时直播信息界面的位置，来确定粉色信息界面的去向。
```
- (void)touchesEnded:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    
//    水平手势判断
    
    if (self.frame.origin.x > Screen_width * 0.2) {
        [UIView animateWithDuration:0.15 animations:^{
            CGRect frame = self.frame;
            frame.origin.x = Screen_width;
            self.frame = frame;
            
        }];
        
    }else{
        [UIView animateWithDuration:0.06 animations:^{
            CGRect frame = self.frame;
            frame.origin.x = 0;
            self.frame = frame;
        }];
    }
    
}

- (void)touchesCancelled:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{

    if (self.frame.origin.x > Screen_width * 0.2) {
        [UIView animateWithDuration:0.15 animations:^{
            CGRect frame = self.frame;
            frame.origin.x = Screen_width;
            self.frame = frame;
            
        }];
        
    }else{
        [UIView animateWithDuration:0.06 animations:^{
            CGRect frame = self.frame;
            frame.origin.x = 0;
            self.frame = frame;
        }];
    }
    
}
```
这是控制器滑动结束时的判定，最终决定了，上下滑动切换频道，左右滑动添加或移除直播信息。
```
- (void)touchesEnded:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
//    NSLog(@"%.2f-----%.2f",livingInfoView.frame.origin.y,Screen_height * 0.8);
    
//    水平滑动判断
//在控制器这边滑动判断如果滑动范围没有超过屏幕的十分之八livingInfoView还是离开屏幕
    if (self.livingInfoView.frame.origin.x > Screen_width * 0.8) {
        [UIView animateWithDuration:0.06 animations:^{
            CGRect frame = self.livingInfoView.frame;
            frame.origin.x = Screen_width;
            self.livingInfoView.frame = frame;
        }];
        
    }else{//否则则回到屏幕0点
        [UIView animateWithDuration:0.2 animations:^{
            CGRect frame = self.livingInfoView.frame;
            frame.origin.x = 0;
            self.livingInfoView.frame = frame;
            
        }];
    }
    
//    上下滑动判断
    if (self.playLayer.frame.origin.y > Screen_height * 0.2) {
//        切换到下一频道
        self.playLayer.image = [UIImage imageNamed:@"PlayView2"];
        
    }else if (self.playLayer.frame.origin.y < - Screen_height * 0.2){
//        切换到上一频道
         self.playLayer.image = [UIImage imageNamed:@"PlayView"];
        
    }
//        回到原始位置等待界面重新加载
    CGRect frame = self.playLayer.frame;
    frame.origin.y = 0;
    self.playLayer.frame = frame;
    
}

- (void)touchesCancelled:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{
    
    //    水平滑动判断
    //在控制器这边滑动判断如果滑动范围没有超过屏幕的十分之八livingInfoView还是离开屏幕
    if (self.livingInfoView.frame.origin.x > Screen_width * 0.8) {
        [UIView animateWithDuration:0.06 animations:^{
            CGRect frame = self.livingInfoView.frame;
            frame.origin.x = Screen_width;
            self.livingInfoView.frame = frame;
        }];
        
    }else{//否则则回到屏幕0点
        [UIView animateWithDuration:0.2 animations:^{
            CGRect frame = self.livingInfoView.frame;
            frame.origin.x = 0;
            self.livingInfoView.frame = frame;
            
        }];
    }
    
    //    上下滑动判断
    if (self.livingInfoView.frame.origin.y > Screen_height * 0.2) {
        //        切换到下一频道，重新加载界面，这里用切换图片做演示。
        self.livingInfoView.backgroundColor = [UIColor colorWithRed:arc4random_uniform(256) / 255.0 green:arc4random_uniform(256) / 255.0 blue:arc4random_uniform(256) / 255.0 alpha:0.5];
        
    }else if (self.livingInfoView.frame.origin.y < - Screen_height * 0.2){
        //        切换到上一频道，重新加载界面，这里用切换图片做演示。
        self.livingInfoView.backgroundColor = [UIColor colorWithRed:arc4random_uniform(256) / 255.0 green:arc4random_uniform(256) / 255.0 blue:arc4random_uniform(256) / 255.0 alpha:0.5];
        
    }
    //        回到原始位置等待界面重新加载
    CGRect frame = self.livingInfoView.frame;
    frame.origin.y = 0;
    self.livingInfoView.frame = frame;
}
```
具体代码下载GitHub:[https://github.com/D-james/SlidPageDemo](https://github.com/D-james/SlidPageDemo)