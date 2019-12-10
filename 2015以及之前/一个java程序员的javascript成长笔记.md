大概很难会想到，自己会系统深入的学习javascript。

曾经作为一个java游戏服务器开发者，虽说后来转行写java web之后，开始写写javascript，但也只是作为补充，做一些修修补补的逻辑。

去年的产品，本来是web端的项目；却因为客户的不断反馈和投诉，被迫用c#做了一个壳子，嵌了游览器，游览器再打开我们的产品的网址；一个意外的举措，发现这玩意简直和c#的winform做的没有两样啊，当然也顺利的蒙骗了客户。

去年年底的项目正式开做的时候，我们真正要做一个客户端的时候；开始也是在c#的winform 和 h5+js之间犹豫不决，但是因为迟迟招聘不到C#客户端开发人员，而项目工期越来越近，所以又一次启动了C#winform壳子+内嵌游览器+h5+js的模式。

当然这次和上次还是有很多不同的，
比如    客户端和服务器之间没有sessioin和cookie的连接。所以很多数据也就存在了客户端。
比如    js所承担的部分，不再是修修补补，而是一个完整的客户端逻辑，大概更加类似于前些年做页游时候，客户端的actionscript所承担的角色。

当然这篇只是谈谈在深入写的发现的之前一些不好的写js的习惯。和大家探讨下

1： this的指向
```
var Rate = {
    url : "http://xxxxx",
    params : {},
    
    getMember : function(){
          $.post(url,params,function(){
                  console.log(this.params); //一定会报错，以为在这个闭环函数里，this指向的是window对象
          },'json');    
    },
}
```


2：灵活使用jquery，拒绝全局变量

比如，关于会员的列表有两种模式，一种是极简模式，一种是详情模式；用两个按钮控制。

一种方式，设置全局变量，点击极简按钮，设置变量为极简模式；点击详情，设置变量为极简模式；请求数据根据全局变量展示不同的信息。

另一种方式，如果是极简模式，那么在极简模式button上添加active的className，如果是详情模式，则移除极简button上的active的class。请求返回的数据根据active所在的button展示不同的信息。
      
3：灵活使用jquery，拒绝id丛生

命名通常是我们的编程的难题之一，一者通常我们英文没有那么好，二者确实有很多变量或者标签实在太过类似；避免躲过标签id命名的方式就是多用jquery选择器。

4：分离js和html

```
	var htmlStr = "" + "<div class='form-group' id='product-catalog-html-"+catalogNum+"'>"
			+ "<label class='col-md-2 control-label'>"+labelText+"</label>"
			+ "<div class='col-md-4'>"
			+ "<input type='text' value='" + catalog
			+ "' class='form-control' id='product-catalog-name-"
			+ catalogNum
			+ "' name='productCatalogName' placeholder='型号'>"
			+ "</div>" + "<div class='col-md-4'>"
			+ "<div class='input-group'>" + "<input type='tel' value='"
			+ price + "' maxlength='9' class='form-control' id='product-catalog-price-"
			+ catalogNum
			+ "' onKeyPress='var ev = event.which?event.which:window.event.keyCode;if((ev>=48&&ev<=57)||ev==8){return true;}else{return false;};' name='productCatalogPrice' placeholder='价格'> <span class='input-group-addon'>分</span>"
			+ "</div>" + "</div>"
			+ "<div class='col-md-2'>"
			+ "<div class='input-group'>" 
			+"<button type='button' class='btn btn-default pull-right' onclick='javascript:delCatalogFun("+catalogNum+");'>删除</button>"+ "</div>" + "</div>"
			+ "<input type='hidden' id='product-catalog-parentid-"+catalogNum+"' value='"+parentidVal+"' />"
			+ "</div>";
	$("#product-catalog-item").append(htmlStr);
```
大家一定见过这样的代码吧，如果要美工忽然修改了结构或者新增了class，那么居然不是到html去修改，而是要跑到js代码里面来修改，真实见了鬼了。

更好的做法，应该使用js模版引擎，那么代码将会精简到这样子
```
var member = getMember();
$("#updateMemberModal").setTemplateElement("newUserTemplate");
$("#updateMemberModal").processTemplate(member);
```

