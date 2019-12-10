![image.png](https://upload-images.jianshu.io/upload_images/1261094-79e95d95797cc167.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


因为人员少，时间紧的问题，我们的前端由原来的webpack打包变成了原生态引入的方式，不过这样也不能隐藏vue所带来的方便。

当然也踩了一些坑，相比较vue所带来的优势，这些坑也是值得的。

下面就先来说说这些坑，比如我们定义了下面这个组件 button-counter

```
Vue.component('button-counter', {   
    data: function () { 
        return { count: 0 } 
    }, 
    template: '<button v-on:click="count++">
          You clicked me {{ count }} times.</button>'
});
```

习惯于之前的使用方式，我们就这样使用组件了。
```
<button-counter/>
<button-counter/>
<button-counter/>
```

但是实际效果来看，只有第一个button-counter被渲染出来了，后面的两个没有出现，查了好久也没找到原因。

另一个同事在找问题的时候，就多试试了，换成了另外一种使用方式

```
<button-counter></button-counter>
<button-counter></button-counter>
<button-counter></button-counter>
```

三个组件被成功的渲染出来了。
