要找到一段字符串中的网址位置，可以用正则表达式，正则表达式比较麻烦，没想到苹果有自带的API哈哈（研究了一下）：`NSDataDetector`用方法`+ (nullable NSDataDetector *)dataDetectorWithTypes:(NSTextCheckingTypes)checkingTypes error:(NSError **)error;`可以看到在类NSTextCheckingTypes中的NSTextCheckingTypeLink即是检测URL的，当然里面还有很多类型
```
    NSTextCheckingTypeOrthography  // language identification
    NSTextCheckingTypeSpelling   // spell checking
    NSTextCheckingTypeGrammar     // grammar checking
    NSTextCheckingTypeDate          // date/time detection
    NSTextCheckingTypeAddress     // address detection
    NSTextCheckingTypeLink                // link detection
    NSTextCheckingTypeQuote         // smart quotes
    NSTextCheckingTypeDash           // smart dashes
    NSTextCheckingTypeReplacement   // fixed replacements, such as copyright symbol for (c)
    NSTextCheckingTypeCorrection      // autocorrection
    NSTextCheckingTypeRegularExpression            // regular expression matches
    NSTextCheckingTypePhoneNumber        // phone number detection
    NSTextCheckingTypeTransitInformation        // transit (e.g. flight) info detection
```
我只介绍URL的识别，其他的原理都一样的,随便找一个字符串 
```
NSString *webString = @"这不是网址\"http://www.baidu.com\"这是我大百度帝国";
[self needHightText:webString];
```

```
- (void)needHightText:(NSString *)wholeText {

//    点击事件用的YYLabel框架，
    YYLabel *mainLabel = [[YYLabel alloc]initWithFrame:CGRectMake(0, 100, 400, 100)];
    [self.view addSubview:mainLabel];
    
    mainLabel.numberOfLines = 0;
    mainLabel.textColor = [UIColor purpleColor];
    
    NSMutableAttributedString *text = [[NSMutableAttributedString alloc]initWithString:wholeText];
    text.yy_font = [UIFont systemFontOfSize:17];
    NSError *error;
    NSDataDetector *dataDetector=[NSDataDetector dataDetectorWithTypes:NSTextCheckingTypeLink error:&error];
    NSArray *arrayOfAllMatches=[dataDetector matchesInString:wholeText options:NSMatchingReportProgress range:NSMakeRange(0, wholeText.length)];
    //NSMatchingOptions匹配方式也有好多种，我选择NSMatchingReportProgress，一直匹配 
    
    //我们得到一个数组，这个数组中NSTextCheckingResult元素中包含我们要找的URL的range，当然可能找到多个URL，找到相应的URL的位置，用YYlabel的高亮点击事件处理跳转网页
    for (NSTextCheckingResult *match in arrayOfAllMatches)
    {
        //        NSLog(@"%@",NSStringFromRange(match.range));
        [text yy_setTextHighlightRange:match.range//设置点击的位置
                                 color:[UIColor orangeColor]
                       backgroundColor:[UIColor whiteColor]
                             tapAction:^(UIView *containerView, NSAttributedString *text, NSRange range, CGRect rect){
                                 NSLog(@"这里是点击事件");
                                  //跳转用的WKWebView
                                 WKWebView *webView = [[WKWebView alloc] initWithFrame:self.view.bounds];
                                 [self.view addSubview:webView];
                                 [webView loadRequest:[NSURLRequest requestWithURL:[NSURL URLWithString:[wholeText substringWithRange:match.range]]]];
                                 
                             }];
    }
    mainLabel.attributedText = text;
}
```
过程解释看代码中的注释，此方法用到了YYText框架，记得导入，还有系统自带的WebKit也需要导入，代码比较简单就不放github地址了，希望可以帮到别人，有问题欢迎指正