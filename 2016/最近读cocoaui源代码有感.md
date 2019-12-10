上半年为了做一个ios的应用，引入了cocoaui库，主要是用来布局ios界面，发现简化了不少代码和工作量。因为在写第一个ios应用的时候，用的代码布局，在适配4s和6的机型时候，几乎被搞死，大量的约束定义充斥在代码中，惨不忍睹。cocoaui的作者是ssdb的作者ideawu，在微博里面比较活跃，有问题at他一般很快就会有回应。ssdb是一个类似于redis的nosql数据库；像这样一个在客户端和服务器领域都有建树的人还是很少的。我等普普通通的程序员，距离这种大神还是有一些距离，不过不能气馁，了解他们才能接近他们。除了羡慕他们解决问题的能力，还是要学习他们解决问题的思路，以及这种解决了问题还分享的精神。

记得大学的时候，每次学习新的语言的时候，总借用第三方库做一个词典，一个天气预报，计算器或者类似于游览器这样小工具等。比如词典使用百度词典api，游览器使用的webview，不过那个时候真的想过实现一个游览器，自己解析html+css。大学时候做xml的解析还是差很多的，最终也是一行代码也没有写，其实想想有很多xml解析库可以借鉴来使用的。说说cocoaui的思路，在布局ios应用界面的时候，即没有使用xib,也没有使用storyboard；而是借用web的html+css来做ios的界面布局。整体思路就是借用libxml2库去解析的html标签，并且将其转换成对应的uibutton,uilabel,uiimage,uiview，放置到界面中，至于他们在界面的中的位置，则是通过解析css，然后去设置border，margin，padding，width，height，corlor，text；关于css的解析，作者写了两篇文章介绍，http://www.ideawu.net/blog/archives/912.html http://www.ideawu.net/blog/archives/868.html，有兴趣可以拜读一下，思路很清晰。



大家都在说ios的约束布局不适合人理解，可是大部分人还是去适应了。可是cocoaui的作者绕过ios的约束布局，借用web的div+css去实现流式布局，这真是给了我们一个很好的选择，同时也提供了一个解决问题的思路，不要陷入其中，而是跳出来从新选择其他的方式。当然能写cocoaui也是需要很深的技术功底的，比如css的解析。按照这个思路，我们可以从新定义android的布局方式，不知道大家有没有想过android的布局文件是很邋遢的，或者说很啰嗦，很冗余。如果是一个小型应用；xml的配置文件大小几乎相当于整个代码文件的大小了。我们按照cocoaui的思路，也可以定义html+css的方式去布局。

1：在activity里面，引入xml文件。

2：使用dom4j去解析xml中的html标签，并且将这些标签转换成

android的控件，比如input(type=button)的转换成Button，input(type=input)转换为edittext，span转换成textview,div转换成layout。

3：解析css文件，定义控件在界面中的位置，比如遇到margin:010 0

0,就去获取button的LinearLayout.LayoutParams，然后调用setMargins函数去设置button的margin，遇到padding：10 0 0 0，同理；遇到color；就去设置button的backgroundColor就可以；等等，就不一一列举。



这样是不是就从新定义了android的布局问题，我们甚至可以使用引用css的方式，去定义公用的css方式，这样能更能简化布局文件。也就解决了androidxml布局冗余的问题。

当然了，这只是一个思路，talk is cheap，show code！