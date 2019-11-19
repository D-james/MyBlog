

###这几天要实现左划删除的功能，发现网上很多帖子大多出自一人之手，然后都是 copy 的文章，其实都没有那么复杂，只实现一个代理方法就可以了

```
- (void)tableView:(UITableView *)tableView commitEditingStyle:(UITableViewCellEditingStyle)editingStyle forRowAtIndexPath:(NSIndexPath *)indexPath
{
    if (editingStyle == UITableViewCellEditingStyleDelete) {

    // 删除数据源的数据,self.cellData是你自己的数据
    [self.cellData removeObjectAtIndex:indexPath.row];
    // 删除列表中数据
    [tableView deleteRowsAtIndexPaths:@[indexPath] withRowAnimation:UITableViewRowAnimationFade];
      }
    
}
```
###默认删除的文字为 Delete，要改为中文实现
```
- (NSString *)tableView:(UITableView *)tableView titleForDeleteConfirmationButtonForRowAtIndexPath:(NSIndexPath *)indexPath
{
    return @"删除";//默认文字为 Delete
}
```
###下面这两个代理方法不用写也可以，默认就是这样
```
- (UITableViewCellEditingStyle)tableView:(UITableView *)tableView editingStyleForRowAtIndexPath:(NSIndexPath *)indexPath {
    return UITableViewCellEditingStyleDelete;
}

- (BOOL)tableView:(UITableView *)tableView canEditRowAtIndexPath:(NSIndexPath *)indexPath
{
    return YES;
}
```
###如果你报了这个错误：
'Invalid update: invalid number of rows in section 0.  The number of rows contained in an existing section after the update (5) must be equal to the number of rows contained in that section before the update (5), plus or minus the number of rows inserted or deleted from that section (0 inserted, 1 deleted) and plus or minus the number of rows moved into or out of that section (0 moved in, 0 moved out)

###你把代理方法中这两个方法顺序搞混了,先删除数据，再删除 cell
`
[self.cellData removeObjectAtIndex:indexPath.row];这个方法在前
`

`
[tableView deleteRowsAtIndexPaths:@[indexPath] withRowAnimation:UITableViewRowAnimationFade];这个方法在后
`
###还有就是，别2到没设置代理，tableView.delegate = self;
###过两天写自定义编辑多个 cell，选择多个 cell 删除，全选删除，默认选择按钮为蓝色，自定义选择按钮