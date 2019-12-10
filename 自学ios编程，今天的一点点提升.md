我这个半路出家的ios程序，写ios也有段时间了，大大小小写了两个应用了，因为是从android转过来的，而且没有经过系统的学习，所以在写应用的时候，难免有些土方式。最近自己这个应用，可不像再用这些土方法了，今天写的时候，就优化了一个小点。


![69E92DAA-B622-41B5-A9D1-7712051DCF86.png](http://upload-images.jianshu.io/upload_images/1261094-defa65ad56f76e00.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

比如上面这个cell，我们自定义UITableViewCell，我们要处理的一个事件就是点击头像，进入这个用户的主页，
我之前的土方式:
首先是在自定义的Cell里面 存放一个变量：
```
@property (nonatomic,retain) UINavigationController* parent;
```
然后在定义头像图片的点击事件中，我这样处理：
```
-(void)gotoProfile
{
    OtherProfileViewController *viewController = [[OtherProfileViewController alloc] init];
    
    viewController.uid = self.item.userId;
    viewController.hidesBottomBarWhenPushed = YES;
    
    [self.parent pushViewController:viewController animated:YES];
}
```
这样，在cellForRowAtIndexPath里面，需要设置每个Cell实例的UINavigationController。
```
-(UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    PostItemCell * cell = [tableView dequeueReusableCellWithIdentifier:self.cellID forIndexPath:indexPath];
    if(cell == nil)
    {
        cell = [[PostItemCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:self.cellID];
    }
    
    cell.parent = self.controller;
    cell.layer.masksToBounds = YES;
    cell.layer.cornerRadius = 5;
    
    RadioItem *item = [self.arr objectAtIndex:indexPath.row];
    [cell setData:item];
    
    return cell;
    
}
```

现在，我通过响应链，就能获取到自定义UITableViewCell的UIViewController
```
-(UIViewController *)getViewController
{
    for (UIView *next = [self superview]; next; next = next.superview) {  UIResponder *nextResponder = [next nextResponder];
        if ([nextResponder isKindOfClass:[UIViewController class]]) {
            return (UIViewController *)nextResponder;
        }
    }
    return nil;
}
```

今天还遇到的一个问题就是NsNull，我之前以为oc的nil和C++/java的NULL一样，最后发现，还是NsNull，下面的方法可以判断NsNull
```
if((NSNull *)item.name == [NSNull null])
```
提升虽然很小，但是很高兴。
