# 移动端Web开发那些坑

## 引言
最近笔者在进行一项移动端Web开发的工作遇到了一些移动端特有的问题，并且其他部门的同事在开发移动端Web遇到了一些问题也向笔者进行求助。为此，笔者特地将最近遇到的一些问题和解决方案总结出来，供以后进行参考。

> 本篇文章着重对遇到的坑及解决方法进行描述，涉及到的原理请读者自行Google。


## 坑1 移动端屏幕适配
在移动端开发的时候我们拿到的设计图往往是以ip6为设计原型的。众所周知，ip6的屏幕是750\*1334，dpr为2。也就是说设计稿上一个宽度为w的元素的px值应该为w/2px。但是我们如果在开发过程中将这个元素的宽设为w/2px，那么在ip6 plus上就会出现该元素的视觉宽度没有ip6上宽的现象，这是因为ip6 plus的dpr为3， 且像素比是1242\*2208 。在以往的开发经验中为实现移动端的适配，那么很多弹性大小的元素都设成了百分比的形式。但是这样做的缺点就是开发的时候特别麻烦，因为UI在切图的时候不可能告诉你每个元素的长宽的百分比，所有的百分比需要开发自己计算或者近似估计。  
这个时候为了更好更方便的适配屏幕，就需要用到rem这个东西。rem是指相对于根元素（即html元素）的字体大小的单位。对于ip6的屏幕来说，如果我们将html的font-size设为100px，那么宽度为w像素的rem值就为w/200rem。那么我们以ip6的设计稿为基础，动态将屏幕宽度比上375（ip6屏幕px值）与100的商，那么页面上所有弹性元素的大小都可以设置为w/200rem。这样就解决了屏幕适配的问题。
```javascript
	document.documentElement.style.fontSize = window.innerWidth / 3.75 + 'px';
    window.onresize = function () {
       document.documentElement.style.fontSize = window.innerWidth / 3.75 + 'px';
    }
```  
在html的head标签里引入上述代码，并且meta标签设置为
```
<meta name="viewport" content="width=device-width,initial-scale=1,user-scalable=0">
```  
这样就能愉快的在开发过程中设置弹性元素的大小了。当然做到了这点还不够，因为在开发的过程中仍然要进行除200的换算。这个时候可以引入postcss-px2rem这个插件（支持gulp/webpack）并设置`px2rem({remUnit: 100})`，这样开发的过程中就可以直接除2然后标上px值，构建的过程中插件会自动将px转换为对应的rem值。  
这个只是一个简单的方案，若要进行更深层次的适配可以参考阿里的`flexible`方案，不过上述方案足够用了。  
不过笔者发现在部分安卓手机中竟然不支持伪元素中使用rem，所以为适配，在伪元素中还是尽量写px。

## 坑2 1px线
目前越来越多的手机屏幕的分辨率越来越高，dpr一般为2或者3。dpr为是默认缩放为100%的情况下，设备像素和CSS像素的比值。那么在这个条件下，css样式中1px的线在显示的时候效果就是2px甚至3px了。这也是为什么我们行内99%的移动端网页的边框那么丑的原因。那么如何避开这个坑呢？1px的解决方案有很多，目前笔者自己采用的方案是用伪元素构造1px的元素然后根据dpr进行50%或者33.4%的缩放。如下所示： 
```css
.setOnePxBySide(@direction:left;@color:#C7C7C7) when (@direction = top) {
  position: relative;
  &:after {
    content: " ";
    position: absolute;
    left: 0;
    top: 0;
    right: 0;
    height: 1px;/*no*/
    border-top: 1px solid @color;/*no*/
    color: @color;
    transform-origin: 0 0;
    transform: scaleY(0.5);
    box-sizing: border-box;
  }
}
```
上述代码是基于less的一个设置top边框1px的函数，读者可以根据需要自行将该代码转换为css形式。

## 坑3 flex布局
flex布局为弹性盒布局，目前已经成为移动端布局最为常用的布局。但是因为flex布局为css3标准且分为旧标准和新标准两个版本，所以在用的时候不仅要写新标准的代码，为了兼容还要写旧标准的代码，这样在开发的时候的就十分的蛋疼，谁也不可能把新旧两个标准都记住而且写的时候同时写两套标准的css代码，这个时候可以使用postcss-autoprefixer插件，这个插件在构建的过程中能够自动将  
```
{
	display: flex;
	flex-direction: column;
	justify-content: center;
}
```  
自动构建为  
```
{
	display: -webkit-box;
    display: -webkit-flex;
    display: -ms-flexbox;
    display: flex;
    -webkit-box-orient: vertical;
    -webkit-box-direction: normal;
    -webkit-flex-direction: column;
    -ms-flex-direction: column;
    flex-direction: column;
    -webkit-box-pack: center;
    -webkit-justify-content: center;
    -ms-flex-pack: center;
    justify-content: center;
}
```  
不过在使用flex进行布局的时候还需要注意一点是安卓4.4以下的部分机型（三星等）不支持换行这个属性，所以如果要兼容4.4以下而且还使用带换行属性的话，还是老老实实将flex布局转为float布局来写吧。  

## 坑4 各种polyfill
- iOS中存在click事件300ms延迟，需要引入fastclick.js；坑爹的是部分安卓机型引入fastclick会有问题，所以要分系统动态引入；
- 如果异步请求方案采用了fetch需要引入whatwg-fetch作polyfill，同时为了使用promise还必须要引入es6-promise做polyfill；
- 如果要做国际化，低版本Safari没有Intl对象，需要引入Intl.js做polyfill；
- 如果用到了object.assign等高级api，还需要引入core-js做polyfill;
- 等等等一切仍未遇到的坑。。。


## 坑5 性能
移动端不像PC端想怎么来就怎么来，由于移动端设备性能的原因很多时候在PC端看起来很美好的东西放到移动端就卡出翔，所以移动端性能很重要，具体表现为：   
- 动画效果尽量用transform translate3d来开启GPU加速，避免使用对top height等变化，以免造成重绘。也就是说移动端动画尽量用translate，scale，rotate及opacity来避免重绘；
- 使用-webkit-overflow-scrolling: touch来提高滑动性能；
- 使用absolute使动画元素脱离文档流，避免对周围元素的干扰；
- 设置backface-visibility为hidden来告诉浏览器不要渲染背向用户的一面；
- 尽可能少的使用box-shadows与gradients；
- 如果性能还有问题那么就不要用动画。。。。


## 后记
不知不觉已经写了这么多坑了，当然，笔者也只是在自己遇到的情况上做的总结，还没有遇到的坑估计已经排成一排等着笔者跳了。这里给大家提个小建议，在开发的过程中最好拿个安卓4.4以下的低端手机做开发测试，如果你的程序在该手机上运行没有问题的话大概率其他手机也不会遇到兼容性问题了。
