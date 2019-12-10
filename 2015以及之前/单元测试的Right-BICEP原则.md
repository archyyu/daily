##简介
对一个函数进行全面的单元测试并非轻而易举。但是如果你是一个老手，你可能知道他会在哪些情况下出错，然后用单元测试去验证。但是新手怎么办，如何去对函数进行一个全面的单元测试呢；幸运的是，这里有一些可以遵循的原则。

那就是我们Right-BICEP原则
Right: 结果是否正确
B:Boundary,边界检测是否正确
I:inverse,调整顺序是否正确
C:cross-check：环形测试
E:error-condition： 异常情况
p:performance : 性能曲线

##结果是否正确
最直接最简单的测试是查看结果是否正确

如果结果正确，我们如何得知



###使用文件数据

如果需要测试的数据非常庞大并且复杂，那么我们可以借助文件来做。举个例子，我们测试Largest.largest(new int[]{});我们可以用下面格式的文件
```
#
#Simple test
#
9 9 8 7 6
9 7 8 9 6
9 8 7 6 9

#
#Negative number test
#
-7 -7 -8 -9 
-7 -8 -9 -7
-7 -9 -8 -7

#
#Mixture:
#
7 -9 -8 -7 7 6 5
0 -1 -2 -3 -5 -6 -7

#
#Boundary conditions
#
1 1
0 0
22222222 22222222
11111111 11111111

```
格式非常简单，当然你也可以使用xml或者json格式的更加容易读取的方式进行测试；测试程序只需要读取对应的数据，一组一组数据的读取，测试。

如果只是几组数据，我们可能觉得这样做有点小题大做；但是设计到上万行数据，那我们还是用读取文件测试的方式，进行测试更加顺利些。

###边界条件
在上一个测试单元中，我们用到了几个边界条件，把最大值放到第一个或者最后一个，数组中含有负数，数值只有一个元素，空数组。边界是单元测试中最需要考虑的地方，因为这也是bug滋生的地方。下面是几个你需要考虑的边界条件


    1：是否含有混乱的违反常理的字母，比如一个文件的名字，"!@#$#%$^%&
    2：格式错误，比如邮箱地址没有域名 archy_yu@sina.
    3:  空值，比如 0,0.0,"",null
    4:  越界，比如年龄超过120，月份超过12，时间超过24小时
    5：不应该重复的数组中，出现重复值
    6：是否已经就位，或者满足执行条件，比如不能在创建文件之前打印它，不能在网络连接上之前连接数据库。

我们在做单元测试的时候 ，只需要考虑上述六个条件，基本可以将边界情况考虑完整。

###环形测试

一些方法函数需要在环形测试的过程中去验证正确性。比如我们算4的平方根乘以4的平凡根，再和4进行比较。
```
public void testSquareRootUsingInverse(){
    double x = mySquareRoot(4.0);
    assertEquals(4.0,x*x,0.00001);
}
```

例子很简单，但是很实用；
对于数据库相关，我们可以做插入的操作，然后再搜索出来。

###交叉测试对比
我们可以实用不同的方式获得结果，然后再对比结果，就拿求平凡根的方式来看，我们可以这样进行测试
```
public void testSquareRootUsingInverse(){
    double number = 3880900.0;
    double root1 = mySquareRoot(number);
    double root2 = Math.squre(number);
    assertEquals(root2,root1,0.00001);
}
```

###异常情况
在实际的生产环境中，什么情况都有可能发生。硬盘满了，网断了，邮箱地址进入黑名单，程序崩溃。你必须检测在这些情况下，你的代码的运行状况。

下面是我们需要考虑的一些情况
```
  1 内存溢出
  2 空间不足
  3 时钟混乱
  4 网络不可达
```

###性能曲线

我们遇到过这样的情况，我们发布的第一个版本运行良好，但是到第二个版本的时候，运行速度却慢成了狗；这是为什么呢？是我们修改了那些东西吗？没有，原来是情况越来越复杂，流量越来越多，需要处理的数据也越来越多，我们的性能曲线呈指数上升造成的，那么如何避免这样的情况呢，只能去测试性能曲线了。
比如我们要检测，一组网址是否可达；开始呢，只有几个网址要检测；慢慢增长，变成了几百个，再增长变成了几千个，性能如何呢？

```
public void testUrlFilter(){
  Timer timer = new Timer();
  String backurl = "http://www.ascascasc.com";
  
  URLFilter filter = new URLFilter(smallList);
  timer.start();
  filter.check(backurl);
  timer.end();
  
  assertTrue(timer.elapsedTimer() <= 1.0);
  
  URLFilter f = URLFiler(bigList);
  timer.start();
  f.check(backurl);
  timer.end();
  assertTrue(timer.elapsedTimer() <= 2.0);

  URLFilter ff = URLFilter(hugeList);
  timer.start();
  ff.check(backurl);
  timer.end();
  assertTrue(timer.elapsedTimer() <= 3.0);
  

}
```

这样，我们就能检测出，随着数据量的增大，我们性能是一个怎样的增长形式，继而分析出我们是否要优化算法，还是变更需求。
