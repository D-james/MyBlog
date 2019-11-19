我做的是直播消息系统，每当有人评论、送礼物、第一次点赞直播消息列表都要及时显示。就像这样
![消息系统效果](http://upload-images.jianshu.io/upload_images/1737720-a34a227992600ea3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

原来思路是这样的：图中红框是我们的消息列表，是一个tableView，每个消息是一个cell，每次接收到一条消息立刻添加到数据数组中，同时刷新tableView，滚动到底部。
原来的代码是这样写的：
```
   [chatDataArray addObject:chatModel];
   [chatTable reloadData];

   NSIndexPath *myIndexPath = [NSIndexPath indexPathForRow: chatDataArray.count - 1 inSection:0];
    [chatTable selectRowAtIndexPath:myIndexPath animated:YES scrollPosition:UITableViewScrollPositionBottom];
```

但是这样写有一个很大的问题就是：每次接受到消息时添加到数据数组中，同时刷新tableView的时候，整个tableView会闪烁一下，这个问题困扰我好久不知道怎么解决，期间用过很多办法（先隐藏再刷新，异步刷新），踩了很多坑，都解决不了问题，分析闪烁的根源是因为刷新的是整个tableView，想想如果只刷新最后一行就好了，但是局部刷新的方法的前提是这行cell原来是存在的，但是这个直播消息是每次有新消息后有刷新列表，一个新的消息产生后伴随着一个新的一行cell（其实是没产生新的cell，有复用机制，这里打个比方，不要误解）。今天不知道怎么开光了，突然想到tableView还有一个insert的方法，一直不怎么用竟然把她忘了。
解决后的代码如下：
```
[chatDataArray addObject:chatModel];

 NSIndexPath *myIndexPath = [NSIndexPath indexPathForRow: chatDataArray.count - 1 inSection:0];
 [chatTable insertRowAtIndexPath:myIndexPath withRowAnimation:UITableViewRowAnimationNone];
 [chatTable selectRowAtIndexPath:myIndexPath animated:YES scrollPosition:UITableViewScrollPositionBottom];
```
新的接受的消息cell直接插入到最后一行，然后再自动滚动到底部。不闪了，哈哈，so easy！完美解决！请记住整个伟大的方法
`- (void)insertRowsAtIndexPaths:(NSArray<NSIndexPath *> *)indexPaths withRowAnimation:(UITableViewRowAnimation)animation;`
希望这篇文章可以帮到你O(∩_∩)O