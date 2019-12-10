netbeans 是我最喜欢的ide之一，主要用来做php和js的开发；其他喜欢的还有visual studio(用来写.net)，idea(用来写java)。


最近构建的项目中用到了vue，因为前端相对简单，所以没有用webpack打包。

但是发现netbeans对于vue文件支持的很不好，去netbeans官网中也没有发现官方或者第三方插件。

但是呢，vue到底属于html文件，所以就重新设置了一下 netbeans，起码可以支持vue中语法的高亮。

工具(tools)=>选项(option)=>其他(Miscellaneous)=>文件(file)，
新建vue的扩展名，并且把它关联到 HTML files(text/html) 

![image.png](https://upload-images.jianshu.io/upload_images/1261094-9730bd693af131f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


这样，对于vue文件，就支持高亮了；虽然并不完美。

效果图

![image.png](https://upload-images.jianshu.io/upload_images/1261094-a86477179eca23fc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


希望支持vue的插件赶快上线。
