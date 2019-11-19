 ## 一.  'xxx' file not found
1.没有正确引入头文件
不是用一个target(如pod文件)用<>,同一个target用""import。有时需要引入绝对路径，如#import <UMMobClick/MobClick.h>


2.项目中引入相同的文件导致文件冲突，找不到文件

3.库文件找不到
1>搜索库文件，有没有导入到项目中。
2>link binary with libraies查看有没有引入当前库，如果是.m文件需要在complie sources 添加.m文件
![image.png](https://upload-images.jianshu.io/upload_images/1737720-46b00c9d69591984.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3>因为库文件配置路径不对，找不到文件
去buildSetring找到
![image.png](https://upload-images.jianshu.io/upload_images/1737720-dae1442f94db7e35.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这三个选项修改为正确的目录，如果是暴露的头文件找不到需要在header search paths 中添加相应目录


## 二. Use of undeclared identifier ''

Use of undeclared identifier 'PayResp'这个我遇到的，
一般没有导入头文件，或者头文件导入的方式不对，是<>还是“”还是绝对路径，会报这个常见的错误，但是我已经导入了啊，这个因为：
工程中有两个一样的库文件，删除一个，我的是都在pod里导入的，一个是微信自己的，一个是友盟的

## 三. Undefined symbols for architecture arm64:错误

找不到引用的静态库文件
1.Linked Frameworks and Libraries 下没有引入库
2.lib库有重复，需要删除一个，删除哪个呢（一个手动引入，一个pod导入），我的实践是删除pod里的，如果库重复，地址不重复，有时也不会报错
3.lib path地址没有配置，一般当手动引入库文件的时候这个地址会自动添加，如果这个地址下的库，找不到会报warning

## 四. duplicate symbols for architecture arm64
有相同的文件,删除一个

## 五. error: Failed with exit code 1
这个错误非常常见
1.可能是证书错误

## 六. 其他
EXC_BAD_ACCES 坏地址访问,野指针访问

unexpectedly found nil while unwrapping an Optional value
在[解包]一个可选值时发现 nil

reference form：缺少头文件