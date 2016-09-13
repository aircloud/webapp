
### 这里面总结一些自己知道的webapp的坑，当然，随着浏览器的进步，我希望这些坑还是被解决比较好


####1.input placeholder问题

在chrome 模拟移动端调试时，显示的非常正常，但是在真机上，placeholder里面的内容明显靠上，非常的不美观

在国外网站，对这个属性的兼容性处理，那就是不要设计input的line-height或者设置line-height为normal即可，

试了一下，虽然在谷歌模拟调试里稍微偏上，但是在“真机上”正常垂直居中～

####2.line－height

line-height经常用于文字居中，不同手机显示效果不一样。什么鬼～

在chrome模拟器上又是显示得非常完美，但是！Android和IOS又各自‘偏移’了。如果把line-height加1px，iPhone文字就会稍微‘正常显示’，由于我们app的ios用户居多，并且android机型太多，不同机型也会显示不同，所以只能退而求其次了。line-height的兼容问题不太好解决，容器高度越小，显示效果的差距越明显。

解决方案：稍微大一点的高度，最好把line-height设置为高度+1px，两个平台显示都不会太‘奇怪’。

####3.使用rem  （兼容性：ie9+）

原理：浏览器的默认字体高都是16px，未经调整的浏览器在显示1em=16px。

rem则是只相对于根元素的font-size，即只需要设置根元素的font-size，其它元素使用rem单位设置成相应的百分比即可；

一般使用：
```
设置html的font-size为62.5%

html {
    font-size: 62.5%;
}
body {
    font-size: 12px;
    font-size: 1.2rem;
}
p {
    font-size: 14px;
    font-size: 1.4rem;
}
```
####4.实现自定义原生控件的样式

由于select移动端原生样式很丑，但是原生弹出样式是符合我们设计的原则

解决方法：将原本select 设置为透明，z-index设置高～再用一个比较好看的样式‘假装’在表面

####5.移动端使用innerHtml绘制

使用innerHTML绘制大段，之后想获取HTML的ID节点，事实上是获取不到的，这种问题在动态创建DOM会经常发生

这也是一个神器的问题，博主自己写了一个移动端轮播插件，在chrome上浏览非常正常，但到了真机上却显示空白，各种百度，最后才发现这么坑的地方…

解决方案：尝试了很多方法之后，老老实实在页面直接用html结构，如果有更好的方法，也请告诉我。

####6.300ms延迟

这个自己在博客里说到的很多，而且网上也有很多的解决方案。

方案一：禁用缩放

在HTML文档头部包含如下meta标签时：
```
<meta name="viewport" content="user-scalable=no"/>
<meta name="viewport" content="initial-scale=1,maximum-scale=1"/>
```
缺点——就是必须通过完全禁用缩放来达到去掉点击延迟的目的，然而完全禁用缩放并不是我们的初衷，我们只是想禁掉默认的双击缩放行为，这样就不用等待300ms来判断当前操作是否是双击。

方案二：更改默认的视口宽度
```
<meta name="viewport" content="width=device-width"/>
```
如果设置了上述meta标签，那浏览器就可以认为该网站已经对移动端做过了适配和优化，就无需双击缩放操作了。

这个方案相比方案一的好处在于，它没有完全禁用缩放，而只是禁用了浏览器默认的双击缩放行为，但用户仍然可以通过双指缩放操作来缩放页面。

兼容性问题：

对于方案一和方案二，Chrome是率先支持的，Firefox紧随其后，然而令Safari头疼的是，它除了双击缩放还有双击滚动操作，如果采用这种两种方案，那势必连双击滚动也要一起禁用。

####7.点击穿透

这个相当于是上一个问题的衍生问题：

问题常见发生场景： 假如页面上有两个元素A和B。B元素在A元素之上。我们在B元素的touchstart事件上注册了一个回调函数，该回调函数的作用是隐藏B元素。我们发现，当我们点击B元素，B元素被隐藏了，随后，A元素触发了click事件。

这是因为在移动端浏览器，事件执行的顺序是`touchstart > touchend > click`。

而click事件有300ms的延迟，当touchstart事件把B元素隐藏之后，隔了300ms，浏览器触发了click事件，但是此时B元素不见了，所以该事件被派发到了A元素身上。

如果A元素是一个链接，那此时页面就会意外地跳转。

解决思路：

1.不要混用touch和click

2.消耗掉touch之后的click

解决方法：

1.只用touch   把页面内所有click全部换成touch事件（ touchstart 、’touchend’、’tap’），注意：a标签的href也是click，需要换成js的跳转。

2.改动最小——350ms后再隐藏B元素

####8. 虚拟键盘导致 fixed 元素错位

fixed元素一定会伴随虚拟键盘的出现，但是虚拟键盘只是“贴”在了viewport上，表面上不会对dom产生“任何”影响，但是这个时候fixed元素表现却变得怪异起来，会错位。

解决原理：虚拟键盘弹出时将fixed元素设置为static，虚拟键盘消失时候设置回来。

解决方案：由于虚拟键盘出现并未抛出事件，而检测scroll或者resize事件，皆会有一定延迟，会出现闪烁现象。则当前获取焦点元素为文本元素，就将fixed元素设置为static。

####9.移动端手势

手指放在屏幕上：ontouchstart     手指在屏幕上滑动：ontouchmove      手指离开屏幕：ontouchend

原理：

1.在touchstart事件触发时，  记录手指按下的时间startTime，本次滑动的初始位置initialPos。

2.在touchmove事件触发时， 记录当前位置nowPosition（实时移动元素），滑动距离movePosition（当前位置nowPosition与初始位置initialPos的差值），判断正负数再决定是左还是右移动。

3.在touchend事件触发时，   记录手指离开屏幕的时间endTime，获得手指在屏幕上停留的时间（endTime－startTime），滑动距离movePosition

判断是否滑动：

如果停留时间少于300ms，则认为是快速滑动，无论滑动距离是多少，都到下一页
滑动距离与‘容器’  大小进行比较，若超过‘容器’大小的1/3，则到下一页

10.iphone动态生成html元素click失效

这个也是神奇的坑，找了很久资料，也没有很原理的解释。

解决方法：  为绑定click的元素增加css样式   cursor：pointer；


****

参考资料罗列：


1.总结开发实践中遇到的坑

http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651551253&idx=1&sn=278b6e2a7e712f8ebf82bf5780937363&chksm=8025a1d4b75228c2a3d05fac04eab79d8a07a99d671d3faf21122e2a32756fde624a7576715e&scene=1&srcid=0913xKK3Nd78IFYwYNuvHZ4Q#rd
