
推荐使用Sublime Text编辑器，打开Sublime，首先保存为.html文件，输入html:5然后按tab键，会生成HTML的基本架构
```
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
</head>
<body>
	
</body>
</html>
```
`<html></html>`是根标签
 
HTML是由标签组成，标签可以设置自己的属性，属性之间是用空格隔开的

标签分为单标签和双标签
单标签`<br /> <img>`
双标签`<html></html>  <title></title>  <head></head> <body></body> <strong></strong>`
 
/是关闭符，表示一个标签的结束
 
标签关系

1.嵌套关系
2.并列关系
大多数标签都可以互相嵌套，构成各种复杂的布局，有些标签需要固定搭配使用 

####标题标签(h1-h6) 
`<h1></h1>`  到 `<h6></h6>`文字依次变小
 
h1经常用于logo，不要轻易使用  
 
`<p>`和`<br>`区别 
段落标签`<p></p>`段落之前有间距
 
换行标签`<br />`换行之间没有间距
 
水平线标签`<hr />`
 
 
####文本格式化标签
```
<b></b> <strong>加粗</strong>
<i></i>   <em>倾斜</em>
<s>></s> <del>删除线</del>
<u></u>  <ins>下划线</ins>
```
 
####图像标签
```
<img src="img/fire.jpg" width="200" height="200" alt="没图" title="我是一张图片" border=1>
```
宽高只设置一个就会等比缩放，不然不计算同时设置宽高会变形
 
####链接标签
```
<a href="http://www.baidu.com" target="_black">百度</a>target="_black"打开新的空白页
<a href="http://www.sina.com" target="_self">新浪（target默认self）</a>
<a href="#">我的主页（未完成）</a>
```
 
####锚点（跳转到当前界面的指定位置）
```
<a href="#music">音乐生涯</a>指定锚点id为music
<p id="music">会跳转到这里</p>
```
####base标签（放在head标签内）
```
<base href="https://tieba.baidu.com/" target="_black">
```
base可以全局指定baseURL和跳转方式,这样所有的src属性都会拼接base的href属性的地址（img，audio，video，a）

####无序列表(前边没有序号，有一个黑点)
```
<h5>我喜欢的水果：</h5>
<ul>
<li>苹果</li>
<li>桃子</li>
<li>梨</li>
<li>西瓜</li>
<li>凤梨</li>
<li>香蕉</li>
</ul>
```

ul标签里面建议只放li标签，但是li标签可以放其他标签，li标签相当于一个容器
 
 
####有序列表(前面会有序号，不常用)
```
<ol>
<li>中国</li>
<li>美国</li>
<li>巴西</li>
<li>日本</li>
</ol>
``` 
 
####自定义列表
```
<dl>
<dt>雍正</dt>
<dd>甄嬛</dd>
<dd>皇后</dd>
<dd>妃子</dd>
</dl>
```

####表格 
```
<!-- 表格 -->
 
<table width="500" height="150" border="1" cellspacing="10" cellpadding="5" align="center">
<thead>
<tr>
<th>name</th>
<th>gender</th>
<th>age</th>
</tr>
</thead>
 
 
<tbody>
<tr>
<td>鸣人</td>
<td>男</td>
<td>18</td>
</tr>
<tr>
<td>佐助</td>
<td>男</td>
<td>19</td>
</tr>
 
</tbody>
 
</table>
```
table里面只能放tr，tr里面只能放td，但td里面可以放别的标签 ，th表头
 
``` 
<!-- input按钮各种type -->
用户名：<input type="text" value="默认值">
<br/>
密  码：<input type="password" maxlength="6"> <!-- maxlength最大输入长度 -->
<br/>
性  别：<input type="radio" name="sex" checked="checked">男 <!-- 默认选中属性checked -->
<input type="radio" name="sex">女<!-- 单选框 需要属性name名字相同-->
<br/>
爱  好：<input type="checkbox" name="">足球 <input type="checkbox" name="">篮球 <input type="checkbox" name="">乒乓球 <!-- 复选框 -->
 
<br/>
 
<input type="button" name="" value="按钮">
<input type="submit" name="">
<input type="reset" name="">
<br/>
<input type="image" name="" src="img/fire.jpg">
<br/>
<input type="file" name="">
```
label标签(扩大包裹的标签点击响应范围)
```
<label>输入账号：<input type="text" name=""></label>
<label for="pw">输入账号：<input type="text" name=""><input type="text" id="pw"></label> 用for 和 id 匹配标记响应的对象
```
 
####下拉菜单
```
选择省份
<select>
<option>北京</option>
<option>上海</option>
<option>深圳</option>
<option selected="selected">太阳</option> 默认选中
</select>
 
选择城市
<select>
<option>请选择城市</option>
<option>大连</option>
<option>石家庄</option>
<option>焦作</option>
<option selected="selected">周口</option>
</select>
```
 
####datalist 和input连用
```
<input type="text" value="输入明星" list="star">
<datalist id="star">
<option>刘德华</option>
<option>刘少华</option>
<option>刘大华</option>
<option>刘中华</option>
<option>刘小华</option>
</datalist>
```

####fieldset和legend连用
```
<fieldset>
<legend>用户登录</legend>
用户登录：<input type="text">
密         码：<input type="password">
</fieldset> 
```
其他标签
header：定义文档的页眉 头部
nav：定义导航链接的部分
foot：定义页脚 底部
article：定义文章
section：定义文档的一节
aside：定义侧边栏
 
<br/>
####input新增属性 

required必填项
placehold占位符
mutiple多选上传
autocomplete 记录上次输入的内容,必要条件:1.form 2.name 3.submit

``` 
<form>
你的名字：<input type="text" name="fuckEgg" autocomplete="on">
<input type="submit" name="">
</form>
```
 
####引用网络视频的两种方式
```
<iframe height=498 width=510 src='' frameborder=0 'allowfullscreen'></iframe>
```
iframe可以直接播放，embed需要使用flash插件,
embed就是插件的意思，可以插入各种东西，不只是视频
```
<embed src="" allowFullScreen="true" quality="high" width="480" height="350" align="middle" allowScriptAccess="always" type="application/x-shockwave-flash"></embed>
```
 
####播放声音
```
	<audio src="music.mp3" controls="controls" autoplay="autoplay"></audio>

	<audio>
		<source src="music.mp3" type="audio/mp3">
		<source src="music.ogg" type="audio/ogg">
		没有支持的音频格式播放
	</audio>
```
autoplay自动播放，默认false，controls是否显示播放器 
####播放视频
```
	<video src="123.mp4" width="200" autoplay controls ></video>
	<video>
		<source src="123.mp4" type="audio/mp4">
		<source src="123.ogg" type="audio/ogg">
		没有支持的视频格式播放
	</video>
```
