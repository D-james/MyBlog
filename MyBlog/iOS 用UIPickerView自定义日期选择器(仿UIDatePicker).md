UIDatePicker只有这四种模式：
```
typedef NS_ENUM(NSInteger, UIDatePickerMode) {
    UIDatePickerModeTime,           // Displays hour, minute, and optionally AM/PM designation depending on the locale setting (e.g. 6 | 53 | PM)
    UIDatePickerModeDate,           // Displays month, day, and year depending on the locale setting (e.g. November | 15 | 2007)
    UIDatePickerModeDateAndTime,    // Displays date, hour, minute, and optionally AM/PM designation depending on the locale setting (e.g. Wed Nov 15 | 6 | 53 | PM)
    UIDatePickerModeCountDownTimer, // Displays hour and minute (e.g. 1 | 53)
}
```
效果图：
只显示日期小时
![只显示日期小时](https://upload-images.jianshu.io/upload_images/1737720-a2902495850a8e39.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
只显示日期小时分钟
![只显示日期小时分钟](https://upload-images.jianshu.io/upload_images/1737720-0ed6ef73b7ae1c23.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
只显示日期小时半点
![只显示日期小时半点](https://upload-images.jianshu.io/upload_images/1737720-9420c66a691d0680.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


如果只显示日期，小时，不显示分钟就得自定义，自定义当然是用UIPickerView，UIPickerView和tableview用法类似，必须实现两个DataSource方法
```
@protocol UIPickerViewDataSource<NSObject>
@required

// returns the number of 'columns' to display.
- (NSInteger)numberOfComponentsInPickerView:(UIPickerView *)pickerView;

// returns the # of rows in each component..
- (NSInteger)pickerView:(UIPickerView *)pickerView numberOfRowsInComponent:(NSInteger)component;
```

介绍一下
```
@protocol UIPickerViewDelegate<NSObject>
@optional

// 指定每个列的宽
- (CGFloat)pickerView:(UIPickerView *)pickerView widthForComponent:(NSInteger)component 

//指定每一列的行号
- (CGFloat)pickerView:(UIPickerView *)pickerView rowHeightForComponent:(NSInteger)component 

//返回指定列指定行的字符串（一般数据源是一个二维数组）
- (nullable NSString *)pickerView:(UIPickerView *)pickerView titleForRow:(NSInteger)row forComponent:(NSInteger)component __TVOS_PROHIBITED;

//返回指定列指定行的格式化字符串
- (nullable NSAttributedString *)pickerView:(UIPickerView *)pickerView attributedTitleForRow:(NSInteger)row forComponent:(NSInteger)component NS_AVAILABLE_IOS(6_0) __TVOS_PROHIBITED; // attributed title is favored if both methods are implemented

//返回指定列指定行的view
- (UIView *)pickerView:(UIPickerView *)pickerView viewForRow:(NSInteger)row forComponent:(NSInteger)component reusingView:(nullable UIView *)view __TVOS_PROHIBITED;

//只要滚动UIPickerView就会调用这个方法,可以通过这个方法拿到滚动的数据，还可以在重组数据，滚动到特定位置对数据进行刷新
- (void)pickerView:(UIPickerView *)pickerView didSelectRow:(NSInteger)row inComponent:(NSInteger)component __TVOS_PROHIBITED;
```

自定义日期选择器的难点是组织数据和滚动时限制数据的显示，重新组织数据
在viewModel中组织好几个数据模型
日期：
```
#pragma mark - 日期数据源方法
- (NSMutableArray *)setUpDataStartDate:(NSDate *)startDate endDate:(NSDate *)endDate {
    
//    如果现在是晚上23：30以后那么日期要加30minute
    if ([self nowHourEqual23] && [self nowMinuteBig30] && self.dateType == D_YearMouthDayHourHalfMinute) {
        startDate = [NSDate dateWithTimeIntervalSince1970:([NSDate date].timeIntervalSince1970+60*30)];
    }
    
    NSTimeInterval timeInterval = [endDate timeIntervalSinceDate:startDate];
    if (timeInterval < 0) {
        return nil;
    }
    
    //    计算开始结束日期之间相差几个月
    NSInteger monthInterval;
    
    NSDateComponents *startComponents = [startDate getDateComponents];
    NSDateComponents *endComponents = [endDate getDateComponents];
    
    NSInteger startYear = startComponents.year;
    NSInteger endYear = endComponents.year;
    
    NSInteger startMonth = startComponents.month;
    NSInteger endMonth = endComponents.month;
    
    monthInterval = (endYear - startYear) * 12 + (endMonth - startMonth) + 1;
    
    
    NSMutableArray *dateArray = [NSMutableArray new];
    
    
    for (int i = 0; i < monthInterval; i++) {
        
        
        NSDateComponents *newComponent = [NSDateComponents new];
        NSInteger nowYear = startYear + (startMonth + i)/12;
        newComponent.year = nowYear;
        
        NSInteger monthYu = (startMonth + i) % 12;
        NSInteger nowMonth = monthYu ? monthYu : 12;
        newComponent.month = nowMonth;
        newComponent.day = 1;
        
        NSDate *newDate = [[NSCalendar currentCalendar]dateFromComponents:newComponent];
        
        //        本月第一天是周几
//        NSInteger weekFirstDay = [newDate getFirstWeekInMonth];
        
        
        //        添加有数据的数据模型
        NSInteger daysMonth = [newDate getDayNumOfMouth];
        
        for (int k = 1; k <= daysMonth ; k++) {

            if (i == 0) {
                if (k < startComponents.day) {
                    continue;
                }
            }
            
            NSString *dateStr = [NSString stringWithFormat:@"%d年%d月%d日",nowYear,nowMonth,k];
            
            [dateArray addObject:dateStr];
        }
        
    }
    
    return dateArray;
}
```

其他数据源方法类似，根据定义的数据模型可以随意组合成各种各样的日期选择器。

显示半点的逻辑有点复杂：
 如果是分钟只显示半点和整点，也就是显示0和30。根据当前时间，就要判断以下几点：
 1.当前日期，时间的分钟数>0且<30那么分钟只显示30
 2.如果大于30，那么小时数也要+1
 3.如果小时数+1的时候是23点，那么日期天数要+1
 
初始化加载数据源的时候需要判断，滚动的时候也要判断
 只有当前日期（今天）有这个判断，其他日期不会

用法：
```
    CustomDatePickerView *pickerV = [[CustomDatePickerView alloc]initWithFrame:CGRectMake(0, self.view.bounds.size.height-400, self.view.bounds.size.width, 400) withType:D_MouthDayHour];
```
初始化的时候要指定类型
```
typedef enum : NSUInteger {
    D_YearMouthDayHourHalfMinute,//日期，小时，半点
    D_MouthDayHour,//日期，小时
    D_MouthDayHourMinute,//日期，小时，分钟
} DatePickerType;
```

代码地址：[https://github.com/D-james/D-CustomDatePickerView](https://github.com/D-james/D-CustomDatePickerView)
