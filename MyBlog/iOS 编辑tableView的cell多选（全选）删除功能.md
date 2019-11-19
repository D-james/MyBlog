###本期带来 tableview 的编辑模式，多选删除、全选删除,统计选中删除数功能。

![
![Snip20160814_2.png](http://upload-images.jianshu.io/upload_images/1737720-0e07c6f843f478af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](http://upload-images.jianshu.io/upload_images/1737720-9984f30951895136.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

实现下面代理方法可以调用系统的选择 cell 左边的蓝色选中图标，当然必须是`tableView.editing = YES;`
```
-(UITableViewCellEditingStyle)tableView:(UITableView *)tableView editingStyleForRowAtIndexPath:(NSIndexPath *)indexPath
{    
    return UITableViewCellEditingStyleDelete | UITableViewCellEditingStyleInsert;   
}
```
删除 cell 用一个中介数组变量来进行统计要删除的 cell 
选择 cell 用方法
`
[self.tableView selectRowAtIndexPath:indexPath animated:NO scrollPosition:UITableViewScrollPositionBottom];
`
注意要用UITableViewScrollPositionBottom，这样才能从头选到尾，不会出现只选择屏幕显示的区域

##具体代码
```

#import "ViewController.h"
#import "Masonry.h"
#import "UIButton+Extention.h"

@interface ViewController ()<UITableViewDelegate,UITableViewDataSource>

@property (weak, nonatomic) UITableView *tableView;
@property (weak, nonatomic) UIButton *selectedBtn;
@property (weak, nonatomic) UIButton *deleteBtn;
@property (weak, nonatomic) UIImageView *editView;
@property (strong, nonatomic) NSMutableArray *dataArr;
@property (strong, nonatomic) NSMutableArray *deleteArr;
@property (assign, nonatomic) NSInteger deleteNum;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    self.deleteArr = [[NSMutableArray alloc]init];
    
    UITableView *tableView = [[UITableView alloc]initWithFrame:CGRectMake(0, 0, self.view.bounds.size.width, self.view.bounds.size.height - 70)];
    [self.view addSubview:tableView];
    self.tableView = tableView;
    
    tableView.delegate = self;
    tableView.dataSource = self;
    
    tableView.editing = YES;
    
    //    编辑区域
    UIImageView *editView = [[UIImageView alloc]init];
    [self.view addSubview:editView];
    self.editView = editView;
    
    editView.userInteractionEnabled = YES;
    //    editView.hidden = YES;
    editView.image = [UIImage imageNamed:@"MyLivingBtnBackground.png"];
    
    
    [editView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.left.right.bottom.equalTo(self.view);
        make.height.equalTo(@65);
    }];
    
    
    
    //    全选
    UIButton *selectedBtn = [UIButton buttonWithImage:@"AllSelectedBtn" title:@"全选" target:self action:@selector(selectedBtnClick)];
    [editView addSubview:selectedBtn];
    self.selectedBtn = selectedBtn;
    
    [selectedBtn setTitle:@"取消全选" forState:UIControlStateSelected];
    
    [selectedBtn mas_makeConstraints:^(MASConstraintMaker *make) {
        make.size.mas_equalTo(CGSizeMake(140, 45));
        make.centerY.equalTo(editView);
        make.centerX.equalTo(@-80);
    }];
    
    //    删除
    UIButton *deleteBtn = [UIButton buttonWithImage:@"delete_btn" title:@"删除(0)" target:self action:@selector(deleteBtnClick)];
    [editView addSubview:deleteBtn];
    self.deleteBtn = deleteBtn;
    
    [deleteBtn mas_makeConstraints:^(MASConstraintMaker *make) {
        make.size.mas_equalTo(CGSizeMake(140, 45));
        make.centerY.equalTo(editView);
        make.centerX.equalTo(@80);
    }];
}
#pragma mark - 全选按钮被点击
- (void)selectedBtnClick {
    if (!self.selectedBtn.selected) {
        self.selectedBtn.selected = YES;
        
        for (int i = 0; i < self.dataArr.count; i++) {
            
            NSIndexPath *indexPath = [NSIndexPath indexPathForRow:i inSection:0];
            [self.tableView selectRowAtIndexPath:indexPath animated:NO scrollPosition:UITableViewScrollPositionBottom];
        }
        [self.deleteArr addObjectsFromArray:self.dataArr];
        self.deleteNum = self.dataArr.count;
        [self.deleteBtn setTitle:[NSString stringWithFormat:@"删除(%lu)",self.deleteNum] forState:UIControlStateNormal];
    }else{
        self.selectedBtn.selected = NO;
        [self.deleteArr removeAllObjects];
        for (int i = 0; i < self.dataArr.count; i++) {
            
            NSIndexPath *indexPath = [NSIndexPath indexPathForRow:i inSection:0];
            [self.tableView deselectRowAtIndexPath:indexPath animated:NO];
            //            cell.selected = NO;
        }
        self.deleteNum = 0;
        [self.deleteBtn setTitle:[NSString stringWithFormat:@"删除(%lu)",self.deleteNum] forState:UIControlStateNormal];
    }
}

#pragma mark - 删除按钮
- (void)deleteBtnClick {
    if (self.tableView.editing) {
        //删除

        [self.dataArr removeObjectsInArray:self.deleteArr];
        [self.tableView reloadData];
        
        self.deleteNum = 0;
        [self.deleteBtn setTitle:[NSString stringWithFormat:@"删除(%lu)",self.deleteNum] forState:UIControlStateNormal];
        
        self.selectedBtn.selected = NO;
        //            你的网络请求
        
        
    }
}


#pragma mark - DataSource
- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
    return self.dataArr.count;
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    static NSString *cellID = @"cellID";
    
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:cellID];
    
    if (cell == nil) {
        cell = [[UITableViewCell alloc]initWithStyle:UITableViewCellStyleDefault reuseIdentifier:cellID];
    }
    
    cell.textLabel.text = [NSString stringWithFormat:@"%@",self.dataArr[indexPath.row]];

    return cell;
}

#pragma mark - tableViewDelegate

-(UITableViewCellEditingStyle)tableView:(UITableView *)tableView editingStyleForRowAtIndexPath:(NSIndexPath *)indexPath
{
    
    return UITableViewCellEditingStyleDelete | UITableViewCellEditingStyleInsert;
    
    
}

- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath {
    [self.deleteArr addObject:[self.dataArr objectAtIndex:indexPath.row]];
    self.deleteNum += 1;
    [self.deleteBtn setTitle:[NSString stringWithFormat:@"删除(%lu)",self.deleteNum] forState:UIControlStateNormal];
    
}

- (void)tableView:(UITableView *)tableView didDeselectRowAtIndexPath:(NSIndexPath *)indexPath {
    [self.deleteArr removeObject:[self.dataArr objectAtIndex:indexPath.row]];
    self.deleteNum -= 1;
    [self.deleteBtn setTitle:[NSString stringWithFormat:@"删除(%lu)",self.deleteNum] forState:UIControlStateNormal];
}

#pragma mark - 懒加载
- (NSMutableArray *)dataArr {
    if (_dataArr == nil) {
        _dataArr = [[NSMutableArray alloc]init];
        for (int i = 0; i < 100; ++i) {
            [_dataArr addObject:@(i)];
        }
    }
    return _dataArr;
}


@end
```
如果不想用系统自带的蓝色圆圈选中样式，想要修改为自定义的选中样式，可以看我的这篇博文[http://www.jianshu.com/p/678da944f6fb](http://www.jianshu.com/p/678da944f6fb)
##gitHub 地址：[https://github.com/D-james/MultipleSelectCell](https://github.com/D-james/MultipleSelectCell)