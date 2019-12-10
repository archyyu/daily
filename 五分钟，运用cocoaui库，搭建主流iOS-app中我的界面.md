首先介绍一些cocoaui，是国内的一名程序员做的开源的开源系统，目的是为了简化ios布局！官网地址：www.cocoaui.com，github地址：https://github.com/ideawu/cocoaui
　　我们这里使用xml定义布局界面，其实就是传统的html + css定义界面，大部分人都有网页布局的经验，搞ios布局还是很容易入手并且快捷的！我们首先看下我们要做的界面：
![](http://upload-images.jianshu.io/upload_images/1261094-dfa5824b9aa9d44d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我们按照html+css的格式来定义这个界面：　　
```
<div>
    <style>
        .headDiv
        {
        width:100%;
        }
        
        .divStyle{
        width:100%;
        height:auto;
        border-bottom: 1 solid #eee;
        background:#fff;
        vertical-align:middle;
        }
        
        .subdivStyle
        {
        height:auto;
        border: 1 solid #eee;
        height:40px;
        background:#fff;
        }
        
        
        .textStyle{
        float:left;
        height:40px;
        valign:middle;
        }
        
        .btnStyle
        {
        background:#EDA67B;
        width:80%;
        height:50px;
        float:center;
        }
        
    </style>
    
    <div id="headContent" class="headDiv">
        <img id="profileHeader" style="width:80px;height:80px;float:center;margin:10px;" src="default_head.png" />
    </div>
    
    <div id="myWashCar" class="subdivStyle" style="width:50%;height:80px;">
        <img style="margin:10px;width:50px;height:50px;valign:middle;" src="ic_mt_coupon" />
        <span type="text" class="textStyle" >消费卷</span>
    </div>
    <div id="myCoupon" class="subdivStyle" style="width:100%;height:80px;">
        <img style="margin:10px;width:50px;height:50px;valign:middle" src="ic_user_main_favorite.png" />
        <span class="textStyle" style="vertical-align:middle;" >我的收藏</span>
    </div>
    
    <div id="myCar" class="divStyle">
        <img style="margin:10px" src="myfollow.png" />
        <span type="text" class="textStyle">我的订单</span>
        <img style="float:right;margin:10px;" src="ic_arrow.png" />
    </div>
    
    
    <div id="myMsg" class="divStyle">
        <img style="margin:10px" src="mylike.png" />
        <span class="textStyle">我的评价</span>
        <img style="float:right;margin:10px;" src="ic_arrow.png" />
    </div>
    <div id="myVersion" class="divStyle">
        <img style="margin:10px" src="moreitems_version.png" />
        <span type="text" class="textStyle">版本更新</span>
        <img style="float:right;margin:10px;" src="ic_arrow.png" />
    </div>
    
    
</div>
```

将其命名为profile.xml文件放到工程中。格式是不是和普通的html+css界面一模样！支持大部分的html标记和css属性！
然后在ProfileViewController中引入profile.xml文件：代码如下：

```
- (void)viewDidLoad
{
    [super viewDidLoad];
    
    [self initSystemBtn];
    
    
    root = [IView namedView:@"profile.xml"];
    [self addIViewRow:root];
    [self reload];
    
    
    [self initEvent];
}
```
点击头像需要进入修改玩家信息界面，需要监听头像的点击事件:
```
-(void)initEvent
{
    __weak typeof(self) me = self;
    IImage *profileHeader = (IImage *)[root getViewById:@"profileHeader"];
    [profileHeader addEvent:IEventClick handler:^(IEventType type,IView *view){
        [me gotoProfileEdit];
    }];
}
```
是不是很简单就能定义一个界面！
　　补充
　　1：对SdWebImage的支持，IImage(UIImageView的再次封装)中暴露了UIIMageView的接口，可以方便的使用sdWebImage，开始是不支持的，和作者沟通了一下，暴露了这个接口！
　　2：对上拉刷新和下拉加载的支持。有例子为证：http://www.cocoaui.com/docs/api/IRefreshControl
　　3: 对于webview的支持！控件中没有对于webview的支持，如果页面中需要嵌入webview则需要动态创建！
　　4：对于radio和checkbox的支持，目前还不支持，需要动态创建，不过非常easy!
　　5: 由于很多app 都需要微信端，xml文件布局可以直接移植到移动端下面！
