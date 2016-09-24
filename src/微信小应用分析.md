# 微信小程序调研
> Version 1.0 Update  
> - 新建文档

-----------------

## 微信小程序是什么  
> 小程序是一种不需要下载安装即可使用的应用，它实现了应用“触手可及”的梦想，用户扫一扫或者搜一下即可打开应用。也体现了“用完即走”的理念，用户不用关心是否安装太多应用的问题。应用将无处不在，随时可用，但又无需安装卸载。  
> ——  by `张小龙`  

通俗来讲，微信小程序是运行在微信内部的应用程序。与普通App不同的是微信小程序无需安装及卸载，官方声称具备与原生同样的用户体验。目前微信小程序还在内测中，并没有官方文档及工具放出来。`微信官方已于9月24晚放出官方文档及工具，无内测App Id也可以使用`。

## Native or Web ？
Native App和Web App哪个是未来的主流这个命题争论了很多年。Native App的优势是对于系统控件接口和框架的调用能力远高于Web App，Web App的优势是适配多平台能力和快速开发迭代能力远高于Native App。那么微信小程序具体是Native还是Web？  
在微信小程序刚推出来的时候，笔者根据官方宣称的具备原生性能的特点以及小程序的代码形式猜测微信小程序是类似于React Native和Weex这类框架，将用js写的代码转化为Native Code并渲染出来，但是在后来的实例分析中笔者又倾向于微信小程序其实还是运行在微信中的Web App（分析见后文）。目前小程序没有开源，具体运行机制和原理官方也没有进行说明。`所以微信小程序具体是Native还是Web这点仍然存疑`。

## 提供了什么？  
微信小程序是运行在微信内部的应用程序，也就是说小程序生态的基础是微信App。小程序具备的所有的能力是不能超过微信App在安装时声明的系统调用权限的。从这一点看微信小程序开发的自由度是没有前端开发或者Native开发自由度高的，具体小程序能够实现什么功能是完全依赖于微信App提供的能力。  
> 截至目前，微信小程序程序提供了如下：
> - 引入了新的文件格式
> - 配套的开发调试IDE
> - 与微信UI匹配的基础组件
> - 调用底层功能的API  

### 新的文件格式
不同于传统的H5页面开发，微信小程序引入了新的文件格式：  
- WXML（WeiXin Markup Language）
- WXSS   （WeiXin Style Sheets）

#### WXML
WXML（WeiXin Markup Language）是框架设计的一套标签语言，结合微信提供的基础组件及事件系统构建页面的结构。  
与HTML类似，WXML也采用了声明式结构，具体作用和HTML一样构造页面结构。不过从表现形式来看WXML更加类似于XML。WXML提供了`数据绑定、列表渲染、条件渲染、模板、事件`这些能力。  

#### WXSS
WXSS(WeiXin Style Sheets)是一套样式语言，用于描述 WXML 的组件样式，其决定 WXML 的组件应该怎么显示。  
WXSS类似于CSS，都是用来控制页面元素的样式。微信官方称WXSS具备CSS的大部分特性，这点和React Native以及Weex一样，都是CSS的子集，但是具备哪些CSS特性官方并没有公布。除此之外，WXSS还引入了样式单位这个概念。  

#### 开发调试IDE
微信小程序为开发者提供了一套开发调试工具，如下图所示：  
![Alt text](http://od6g4gld9.bkt.clouddn.com/1474654172285.png)  
从使用情况上来看，这套开发调试工具提供了编辑、调试（涵盖Chrome Dev Tool大部分功能）、预览、部署功能，基本能够应付日常的开发工作。  
笔者通过解压IDE发现IDE是通过NW.js + React.js开发完成的。除此之外，笔者还发现了一个名为trans的目录，包含如下文件：  
 - transConfigToPf.js
- transManager.js
- transWxmlToHtml.js
- transWxmlToJs.js
- transWxssToCss.js    

通过文件名我们就能够知道微信小程序将.wxml及.wxcss转为了html和css，此外transConfigToPf是将config转为PageFrame，搜索了下项目发现有个名为pageFrameTpl.js的js文件：  
```javascript
"use strict";
module.exports = '\n<!DOCTYPE html>\n<html lang="zh-CN">\n\n<head>\n  <link href="https://res.wx.qq.com/mpres/htmledition/images/favicon218877.ico" rel="Shortcut Icon">\n  <meta charset="UTF-8" />\n  <meta name="viewport" content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0" />\n\n  <script>\n    var __webviewId__;\n  </script>\n\n  <!-- percodes -->\n\n  <!--{{WAWebview}}-->\n\n  <!--{{reportSDK}}-->\n\n  <!--{{webviewSDK}}-->\n\n  <!--{{exparser}}-->\n\n  <!--{{components_js}}-->\n\n  <!--{{virtual_dom}}-->\n\n  <!--{{components_css}}-->\n\n  <!--{{allWXML}}-->\n\n  <!--{{eruda}}-->\n\n  <!--{{style}}-->\n\n  <!--{{currentstyle}}-->\n\n  <!--{{generateFunc}}-->\n</head>\n\n<body>\n  <div></div>\n</body>\n\n</html>\n';
```  
即使该文件压缩了我们也能看到这个文件的功能是输出一个html文件字符串，其中通过{{xxx}}的形式对各个模块做替换。也就是说很大可能这个模板就是入口html模板。这其中还发现virtual_dom这个字段，说明了微信小程序也是用了虚拟dom的技术。从这个方面来看，微信小程序应该是Web App。由于所有IDE代码均压缩混淆而且本文重点不在IDE，故关于IDE方面的内容便不再详细展开。

#### 微信UI匹配的基础组件

很久之前微信官方团队便开源了WeUI这套UI组件，随着微信小程序的推出WeUI也发布了1.0 Release。微信小程序提供的组件应该就是基于WeUI演进而来的。  
微信小程序的基础组件包括以下八大类：  
-	视图容器(View Container)
	-	view 视图容器
	-	scroll-view 可滚动视图容器
	-	swiper 滑块视图容器
-	基础内容(Basic Content)
	-	icon 图表
	-	text 文字
	-	progress 进度条
-	表单(Form)
	-	button 按钮
	-	form 表单
	-	input 输入框
	-	checkbox 多项选择器
	-	radio 单项选择器
	-	picker 列表选择器
	-	slider 滚动选择器
	-	switch 开关选择器
	-	label 标签
-	操作反馈(Interaction)
	-	action-sheet 上拉菜单
	-	modal 模态提示框
	-	toast 消息提示框
	-	loading 加载提示符
-	导航(Navigation)
	-	navigator 应用链接
-	多媒体(Media)
	-	audio 音频
	-	image 图片
	-	video 视频
-	地图(Map)
	-	map 地图
-	画布(Canvas)  
	-	canvas 画布

### 调用底层功能的API
微信小程序提供了丰富的微信原生API，可以方便的调起微信提供的能力。具体API如下：  
- 网络 API 列表
	- wx.request	发起网络请求
	-	wx.uploadFile	上传文件
	- wx.downloadFile	下载文件
	-	wx.connectSocket	创建 WebSocket 连接
	-	wx.onSocketOpen	监听 WebSocket 打开
	- wx.onSocketError	监听 WebSocket 错误
	- wx.sendSocketMessage	发送 WebSocket 消息
	- wx.onSocketMessage	接受 WebSocket 消息
	- wx.closeSocket	关闭 WebSocket 连接
	- wx.onSocketClose	监听 WebSocket 关闭  
- 媒体 API 列表
	- wx.chooseImage	从相册选择图片，或者拍照
	- wx.previewImage	预览图片
	- wx.startRecord	开始录音
	- wx.stopRecord	结束录音
	- wx.playVoice	播放语音
	- wx.pauseVoice	暂停播放语音
	- wx.stopVoice	结束播放语音
	- wx.getBackgroundAudioPlayerState	获取音乐播放状态
	- wx.playBackgroundAudio	播放音乐
	- wx.pauseBackgroundAudio	暂停播放音乐
	- wx.seekBackgroundAudio	控制音乐播放进度
	- wx.stopBackgroundAudio	停止播放音乐
	- wx.onBackgroundAudioPlay	监听音乐开始播放
	- wx.onBackgroundAudioPause	监听音乐暂停
	- wx.onBackgroundAudioStop	监听音乐结束
	- wx.chooseVideo	从相册选择视频，或者拍摄
	- wx.saveFile	保存文件
- 数据 API 列表
	- wx.getStorage	获取本地数据缓存
	- wx.setStorage	设置本地数据缓存
	- wx.clearStorage	清理本地数据缓存
- 位置 API 列表
	- wx.getLocation	获取当前位置
	- wx.openLocation	打开内置地图
- 设备 API 列表
	- wx.getNetworkType	获取网络类型
	- wx.getSystemInfo	获取系统信息
	- wx.onAccelerometerChange	监听重力感应数据
	- wx.onCompassChange	监听罗盘数据
- 界面 API 列表
	- wx.setNavigationBarTitle	设置当前页面标题
	- wx.showNavigationBarLoading	显示导航条加载动画
	- wx.hideNavigationBarLoading	隐藏导航条加载动画
	- wx.navigateTo	新窗口打开页面
	- wx.redirectTo	原窗口打开页面
	- wx.navigateBack	退回上一个页面
	- wx.createAnimation	动画
	- wx.createContext	创建绘图上下文
	- wx.drawCanvas	绘图
	- wx.hideKeyboard	隐藏键盘
	- wx.stopPullDownRefresh	停止下拉刷新动画
- 开放接口
	- wx.login	登录
	- wx.getUserInfo	获取用户信息
	- wx.requestPayment	发起微信支付  

## 技术分析

### 整体框架
微信小程序提供了自己的视图层描述语言 WXML 和 WXSS，以及基于 JavaScript 的逻辑层框架，并在视图层与逻辑层间提供了数据传输和事件系统。其核心是一个响应的数据绑定系统。微信小程序整个系统分为两块视图层（View）和逻辑层（App Service）。框架通过提供navigator组件及navigateTo、navigateBack、redirectTo来实现整体路由管理及跳转。

### 文件结构
微信小程序包括一个描述整体程序的 app 和多个描述各自页面的 page。  
微信小程序的主体部分即入口部分由app.js，app.json，app.wxss组成，其中：  
-	app.js 提供了入口文件的一些初始化和绑定
-	app.json 提供了项目的结构和一些项目配置（该配置文件应该是为微信的云端编译声明项目需要用到的组件及需要加载的page）
-	app.wxss 全局样式  

微信小程序的业务页面是由四个部分组成，功能与入口文件类似：
-	module.wxml 页面结构 （必须）
-	module.js 页面逻辑 （必须）
- module.wxss 页面样式表（非必须）
- module.json 页面配置 （非必须）  

> `注意`
> module页面的这四个文件必须具有相同的路径与文件名。  

### 逻辑层

#### 程序配置
微信小程序通过app.json文件来进行全局配置，决定页面文件的路径、窗口表现、设置网络超时时间、设置多 tab 等。包含以下配置项：  
- pages		设置页面路径
- window		设置默认页面的窗口表现
- tabBar	设置底部 tab 的表现
- networkTimeout	设置网络超时时间
- debug		设置是否开启 debug 模式  

全局里面的这些配置应该就是对原生的设置，换一句话来讲`微信小程序大概率除了tabBar和导航栏是Native，其他均为Web`。

#### App 入口程序
微信小程序需要在app.js中通过调用`App()`函数进行注册，该函数接受一个 object 参数，其指定小程序的生命周期函数等。整个程序的生命周期包括：  
- onLaunch
- onShow
- onHide  

**onLaunch**是小程序的初始化函数，全局只会触发一次。**onShow**是小程序的显示函数，当小程序启动，或从后台进入前台显示时触发。**onHide**是小程序的隐藏函数，当小程序从前台转向后台时触发。  `根据此声明周期我们可以知道小程序一次打开后会一直存活，即使用户从小程序中退出到微信的主界面。只有在系统资源暂用较高时才会被真正销`。  
App还提供了一个函数App.getCurrentPage()来获取当前页面的实例。除此之外，微信小程序还提供了一个全局函数getApp()用来获取小程序的实例。  

#### Page 实际页面
微信小程序提供Page()函数进行页面的注册。该函数接受一个 OBJECT 参数，其指定页面的初始数据、生命周期函数、事件处理函数等。不同于App，Page的生命周期包括：  
- onLoad		生命周期函数--监听页面加载
- onReady		生命周期函数--监听页面渲染完成
- onShow		生命周期函数--监听页面显示
- onHide	生命周期函数--监听页面隐藏
- onUnload	生命周期函数--监听页面卸载  

![Alt text](http://od6g4gld9.bkt.clouddn.com/1474659000252.png)


Page()函数的入参中还包含`data`关键字，该字段为页面的初始化数据。

##### Page 事件
Page()函数除了初始化数据和生命周期函数，Page 中还可以定义一些特殊的函数：事件处理函数。在渲染层可以在组件中加入事件绑定，当达到触发事件时，就会执行 Page 中定义的事件处理函数。  
小程序中事件分为冒泡事件和非冒泡事件：
-	冒泡事件：当一个组件上的事件被触发后，该事件会向父节点传递。
-	非冒泡事件：当一个组件上的事件被触发后，该事件不会向父节点传递。  

其中冒泡事件包括：  
- touchstart	手指触摸
- touchmove	手指触摸后移动
- touchcancel	手指触摸动作被打断，如来电提醒，弹窗
- touchend	手指触摸动作结束
- tap	手指触摸后离开
- longtap	手指触摸后，超过350ms再离开

`除此之外的其他组件自定义事件都是非冒泡事件`。  

微信小程序的事件绑定分为`bind`和`catch`。bind事件绑定不会阻止冒泡事件向上冒泡，catch事件绑定可以阻止冒泡事件向上冒泡。小程序对事件绑定作如下规定：  
- key 以bind或catch开头，然后跟上事件的类型，如bindtap, catchtouchstart；
- value 是一个字符串，需要在对应的 Page 中定义同名的函数；  

由此可以看出在事件绑定这块微信小程序和现在的前端开发有很大不同，其事件类型应该是现在DOM事件的子集，在绑定这块bind关键字对应on，catch关键字对应on + stopPropagation。  

#####setData()
微信小程序还提供了Page.setData()函数来进行数据的更新。这个和React中的setState十分相似。类似于React，微信小程序不能通过修改this.data来更新数据，必须通过this.setData()来进行数据更新。由此可以看出来微信小程序不是一种双向绑定的结构，setData()函数每次调用后应该会自动调用render进行页面的更新。  

#### 模块化与作用域
微信小程序自身自备了一套模块化方案，通过require函数引入某个模块，通过module.exports对外暴露模块接口。  
微信小程序不同模块之间享有不同的作用域，每个文件声明的变量和函数只在该文件中有效。

### Hello World 实例
这里笔者对微信提供一个Hello World的Demo进行分析。整个Hello World程序效果如下图：  
![Alt text](http://od6g4gld9.bkt.clouddn.com/1474663476561.png)  

Hello World整体项目目录结构如下：  
![Alt text](http://od6g4gld9.bkt.clouddn.com/1474660980627.png)

首先是**app.json**，其中对导航栏以及页面进行了配置。
```json
{
  "pages":[
    "pages/index/index",
    "pages/logs/logs"
  ],
  "window":{
    "backgroundTextStyle":"light",
    "navigationBarBackgroundColor": "#fff",
    "navigationBarTitleText": "WeChat",
    "navigationBarTextStyle":"black"
  }
}
```
从上述配置我们可以看到整个程序包含index和logs两个页面。微信小程序在加载的时候默认加载index页面。**app.wxss**文件设置了全局样式，和css几乎一样，这里不再赘述。我们看下**app.js**：  
```javascript
App({
  onLaunch: function () {
    //调用API从本地缓存中获取数据
    var logs = wx.getStorageSync('logs') || []
    logs.unshift(Date.now())
    wx.setStorageSync('logs', logs)
  },
  getUserInfo:function(cb){
    var that = this;
    if(this.globalData.userInfo){
      typeof cb == "function" && cb(this.globalData.userInfo)
    }else{
      //调用登录接口
      wx.login({
        success: function () {
          wx.getUserInfo({
            success: function (res) {
              that.globalData.userInfo = res.userInfo;
              typeof cb == "function" && cb(that.globalData.userInfo)
            }
          })
        }
      });
    }
  },
  globalData:{
    userInfo:null
  }
})
```
app.js在onLaunch的时候调用wx.setStorageSync储存了初始化完成的时间。app.js还提供了一个getUserInfo方法用来进行登录获取用户信息，并且设置了设置了全局数据globalData，里面放入了用户信息userInfo，初始值为null。  
接下来我们看下index页面，这个页面是该项目在初始化完成后加载的第一个页面。首先看下整体的项目结构**index.wxml**:   
```html
<view class="container">
  <view  bindtap="bindViewTap" class="userinfo">
    <image class="userinfo-avatar" src="{{userInfo.avatarUrl}}" background-size="cover"></image>
    <text class="userinfo-nickname">{{userInfo.nickName}}</text>
  </view>
  <view class="usermotto">
    <text class="user-motto">{{motto}}</text>
  </view>
</view>
```  
这个页面结构很简单，就是包括一个image来显示用户头像、一个text显示用户名及一个text显示名为motto的变量，其中在用户头像及姓名区域绑定了一个tap事件，调用bindViewTap函数。**index.wxss**直接略过，接下来看下**index.js**:  
```javascript
var app = getApp()
Page({
  data: {
    motto: 'Hello World',
    userInfo: {}
  },
  //事件处理函数
  bindViewTap: function() {
    wx.navigateTo({
      url: '../logs/logs'
    })
  },
  onLoad: function () {
    console.log('onLoad')
    var that = this
  	//调用应用实例的方法获取全局数据
    app.getUserInfo(function(userInfo){
      //更新数据
      that.setData({
        userInfo:userInfo
      })
      that.update()
    })
  }
})
```  
index.js首先拿到通过getApp()拿到了程序实例，接下来调用Page函数进行页面注册。在注册函数的参数中index.js设置了两个初始数据userInfo和motto，其中motto等于hello world，与index.wxml中的text对应，userInfo等于null。注册函数的参数中还声明了bindViewTap函数，那么结合前文及该函数我们知道在点击用户区域的时候该页面会调用wx.navigateTo函数路由至logs页面。index.js还在Page的onLoad生命周期里调用app中的getUserInfo函数进行登陆并获取用户信息，然后通过setData函数对userInfo进行更新。那么index.wxml中与userInfo绑定的image和text就会渲染成用户的头像以及用户名。logs页面大同小异，这里不再赘述。  
以上即为Hello World这个应用的全部分析。  


## 结论
- 微信小程序大概率还是Web应用（除了导航，窗体及tabbar），只不过微信对其做了很好的封装；
- 微信小应用提供了一套非常完整的基础组件及丰富的API接口，并且封装效果十分好，能够很大的提升开发人员的开发效率（必须具备一定的前端水平）；
- 微信小程序因为目前并不能跑在微信应用里，所以性能到底如何还是个未知数；
- 微信小程序的开发模式必须完全按照官方的规范来，且和现有的前端框架并不兼容。这个就会带来学习及开发成本（不排除日后官方或社区推出各种XXX转微信小应用的工具）；
- 微信小程序目前还在初始阶段，也无社区支持，遇到无法从官方文档解决的问题就有可能直接组开开发和维护应用；
- `微信小程序是提交代码至云端进行编译且无法加载任何Native的安全控件，这里有很大的安全隐患`

## 引用
【1】：[微信小程序官方文档](https://mp.weixin.qq.com/debug/wxadoc/dev/?t=1474644088715)  
【2】：[微信小程序剖析【下】：运行机制](http://mp.weixin.qq.com/s?__biz=MjM5Mjg4NDMwMA==&mid=2652974093&idx=1&sn=0570a243304ea8bb7d1b636624886fb1#rd)  
【3】：[微信小程序，一个有局限的类似 React Native 轮子！](http://www.jianshu.com/p/060c6f3dd4e8)  
【4】：[gavinkwoe/weapp-ide-crack@github](https://github.com/gavinkwoe/weapp-ide-crack/blob/master/README.md)



 

