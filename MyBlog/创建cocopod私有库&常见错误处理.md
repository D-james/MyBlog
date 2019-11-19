## 概述
cocopod私有库是建立在git基础上的，必须先要有两个git地址:

1.保存cocopod私有库的git地址。里面是你上传的私有库各个版本的podspec文件<br>    
2.将要上传到私有库的项目git地址<br>    
之后根据这两个git地址，与cocopod私有库产生连接，连接的关键就是项目中的.podspec文件<br>

## 创建步骤

1. 创建两个git项目，一个用于存放本地私有库(mySpec)，一个用于存放要上传的项目(demoTool)地址。在github,码云，gitlab创建都可以。<br>    
2. 将pod远程仓库添加到本地：`pod repo add myspec git地址`。（其中myspec是你本地私有库的名字，可以任意。git地址是上一步创建的，你本地私有库对应的git地址）<br>     
3. `git clone`下来用于存放项目的另一个git地址，在里面新建要上传的项目(例如：demoTool)，<br>    
4. `pod spec create demoTool`添加上传私有库的配置文件.podspec（podspec文件名要与项目名相同），会生成demoTool.podspec文件<br>
5. 在项目里面选择正确配置          
	支持设备，支持最低版本，登录证书等，编译一下项目，确保没有error<br><br>    
6. 编辑podspec文件(关键)<br>    
	可以用sublime text编辑<br>
	
```  

Pod::Spec.new do |s|

  #项目名
  s.name         = "Tools"

  #项目版本，这个版本要和git tag 要相同
  s.version      = "3"

  #项目简介，要重新编辑，不然会报warning
  s.summary      = "Tools."

  #项目描述，要比s.summary长，不然会报warning
  s.description  = <<-DESC
                    this is Tools
                   DESC

  #项目主页地址
  s.homepage     = "https://gitee.com/D-James/Tools"

  #许可证   type：协议类型    file:协议文件名 或者另一种写法：s.license = "MIT License"，创建项目的时候选的什么许可证就填什么
  s.license      = { :type => "MIT", :file => "FILE_LICENSE" }

  #作者
  s.author       = "duan"

  #支持最低版本，要与项目中所选一致
  s.platform     = :ios, "8.0"

  #项目git地址，可以为ssh地址，也可以为http地址
  s.source       = { :git => "git@gitee.com:D-James/Tools.git", :tag => s.version.to_s }

  #需要上传到私有库的文件   这个地址是已项目地址为根目录地址。*.{h,m}指的是该目录下的所有.h.m文件
  s.source_files  = "Tools/Tools/**/*.{h,m}"

  #项目所依赖的资源地址，资源包括storyboard,xib，图片
  s.resources  = "Tools/Tools/**/*.{storyboard,xib}", "Tools/Assets.xcassets"

#添加pch文件
	s.prefix_header_contents = '#import "ColorGC.h"','#import "FontSizeGC.h"'
	
  #是否是RAC
  s.requires_arc = true

  #项目所依赖的三方库 依赖多个要写多个 s.dependency
  s.dependency "HLNetworking"


s.libraries     = 'z', 'sqlite3' #表示依赖的系统类库，比如libz.dylib等
s.frameworks    = 'UIKit','AVFoundation' #表示依赖系统的框架
s.ios.vendored_frameworks = 'YJKit/YJKit.framework' # 依赖的第三方/自己的framework
s.vendored_libraries = 'Library/Classes/libWeChatSDK.a' #表示依赖第三方/自己的静态库（比如libWeChatSDK.a）
#依赖的第三方的或者自己的静态库文件必须以lib为前缀进行命名，否则会出现找不到的情况，这一点非常重要
	end
	
```    	 
7.提交上传项目改动
```
	git add .  
	git commit -m "init version 1"  
	git push  
	git tag 1  //tag值要与podsec文件的s.version一致
	pod lib lint // 验证podsepc文件格式，验证项目确保项目能跑通
	git push --tags  //提交tag值
```
8.最后将podspec文件上传到私有库<br>    
	`pod repo push myspec --allow-warnings // myspec是私有pod仓库的名字，--allow-warnings是忽略警告`
	注意：如果这一步没有上传成功，很有可能是你的podspec文件有问题，请仔细检查。
<br><br>

9.调用<br>
	修改podfile文件中，有两种方式：<br>
	    1.在头部添加<br>
	    `source 'https://github.com/CocoaPods/Specs.git'  #官方仓库的地址`<br>
	    `source 'http://git.jsjit.cn/ios/GCModular/ModuleSpec.git'`http地址是你的私有库文件git地址<br>
	    2.pod ‘库名’, :path => '库地址'<br>
建议第一种，只用导入一次（前提是你的私有库地址都关联到source地址）。第二种每个私有库都需要引用地址
<br>

10.Pod私有库的更新<br>
有时别人提交了新的私有库，pod install找不到，需要更新：pod repo update 私有库名字<br>
<br><br>

## 最后关于报错：pod lib lint后的验证报错<br>
1.error: unknown type name<br>
在spec文件添加s.libraries = "c++","c"<br>
然后执行：pod lib lint --allow-warnings --use-libraries<br>

2.file not found with <angled> include; use "quotes" instead<br>
创建自己的库时依赖其它的库，引用的不对。正确的引用是 `#import <YYModel/YYModel.h>`把引用地址写全 <br>

3.添加MRC文件<br>
```
non_arc_files = 'GCTool/GCTool/NetworkTool/Zresponse.pb.h','GCTool/GCTool/NetworkTool/Zresponse.pb.m'

s.exclude_files = non_arc_files

s.subspec 'no-arc' do |sp|

sp.source_files = non_arc_files

sp.requires_arc = false

end
```
4.会上传APPdelegate等无关文件的问题<br>
当推送的库中包含子文件夹的时候，可能会把APPdelegate等s.sourcefile指定路径以外的文件拉取到本地，有时是因为库内引用了APPdelegate，或是APPdelegate的分类，删掉APPdelegate的分类后，拉取库还是会有APPdelegate，这时把sourcefile对应的库的子文件夹依次写出上传，而不要用**，解决这个问题<br>


5.关于更新文件是否需要更改版本号<br>
如果增删文件需要更新版本号<br>
pod lib lint是验证本地文件是否合法，po repo push 是验证远程仓库是否合法，如果本地验证通过而远程验证失败，确保本地文件与远程完全相同，如果已经push,还是远程验证失败，需要更新版本号，增加新的版本，才能和远程同步

6.如果私有库里面有引用了私有库<br>
验证和上传的命令都要加入--sources='http://git.jsjit.cn/ios/GCModular/ModuleSpec.git,https://github.com/CocoaPods/Specs'<br>

7.The 'Pods-App' target has transitive dependencies that include static binaries错误<br>
添加--use-libraries<br><br>

8.Returned an unsuccessful exit code错误<br>
很多情况下会报这个错误，这个错误下面会提示NOTE，NOTE就是具体的验证不通过原因。很可能是你的spec文件配置不对。<br>
1.s.source目录不对。<br>
2.报引用三方库问题。还是添加--use-libraries<br>
有时NOTE报的错误只是警告，找NOTE不完全对，找error也不完全对，全面检查一下spec文件，看看哪里有错，今天就被错误提示误导了<br>

9.添加图片问题<br>
s.resources设置图片的路径，要设置各个图片所在的根目录路径。<br>
如果设置路径的为**，路径下还有子文件夹，将会生成蓝色文件夹，主项目找不到图片，放在asset目录下也会找不到图片，需要在主项目中加载图片方式指明bundleid,或者在主项目再添加图片引用<br>

10.file not found<br>
1)项目编译是否能通过，能通过可能是三方库的spec.dependency没有设置<br>
2)有时报错：一个官方库内部的文件找不到自己的文件。这是因为不能添加整个库的头文件到pch,也就是spec文件中的s.prefix_header_contents的配置，比如我添加了整个Protobuf库的头文件到pch,能编译过，但是上传库就报错google/protobuf/Any.pbobjc.h找不到，所以这种情况不要全局添加，哪里需要再添加到哪里。<br>
3)spec分了很多spec文件夹，每个文件夹又s.depedency很多文件，错误提示很多文件找不到，试试添加--skip-import-validation验证参数

最终命令就成了pod repo push ModuleSpec --allow-warnings --use-libraries --sources='http://git.jsjit.cn/ios/GCModular/ModuleSpec.git,https://github.com/CocoaPods/Specs'<br>
