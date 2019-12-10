[UITableView作为iOS开发最常用的一个控件，如果说对于他的掌握基本代表了ios程序的开发水平，我想也不是太过分。今天要讲的就是如何动态规划UITableViewCell的高度。

首先需要掌握的技术点：

*UITableView的基本使用*
*约束的使用*
*自定义UITableViewCell*

看下我们要做的界面：

![只有单行文字](http://upload-images.jianshu.io/upload_images/1261094-7b60f68c8d0ff3f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![多行文字](http://upload-images.jianshu.io/upload_images/1261094-6c1d590df98c96d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果只有单行问题，那么我们只需要固定承载文字的UILabel的高度就可以了，如果是多行文字，那么我们就需要用约束设置文字和上下图片的距离了。

![设置约束](http://upload-images.jianshu.io/upload_images/1261094-860c01d5c228d1a7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

并且设置UILabel为character wrapper，lines number为0；
这里还不够，因为我们知道，我们在使用UITableView的时候，通过实现一个获取UItableViewCell的高度的函数，那么我们就需要在这里做文章，让他动态的计算出自定义UItableViewCell的高度。

我们看这个自定义的UITableViewCell，除了中间的UILabel，其他的控件都是固定高度的，也就是说我们得到了UILabel的高度，就知道了整个UItableviewcell的高度。 

关键函数:
```
//自定义UItableViewCell
-(void)setData:(id)data
{
    self.item = (RadioItem*)data;
    self.label = self.item.talkContent;
    [self layoutIfNeeded];
    self.item.height = xxx(除去UILabel的高度) + self.label.frame.size.height;
}
//
-(CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath
{
    RadioItem *item = [self.arr objectAtIndex:indexPath.row];
    return item.height;
}
```

希望对大家有所帮助
