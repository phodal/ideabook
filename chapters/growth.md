一份代码打造跨平台应用
===

概况
---

### 背景

Web本身就是跨平台的，这意味着这中间存在着无限的可能性。

我是一名Web Developer，对于我来能用Web开发的事情就用Web来完成就好了——不需要编译，不需要等它编译完。我想到哪我就可以写到哪，我改到哪我就可以发生哪发生了变化。

最近我在写Growth——一个帮助开发人员成长的应用，在近一个月的业余时间里，完成了这个应用的：

 - 移动应用版：Android、Windows Phone、iOS（等账号和上线）
 - Web版
 - 桌面版：Mac OS、Windows、GNU/Linux

### ShowCase

截图合并如下：

![growth-full-platforms.png](./images/growth-full-platforms.jpg)

### Ionic & Electron & Cordova

而更重要的是它们使用了同一份代码——除了对特定设备进行一些处理就没有其他修改。相信全栈的你已经看出来了：

Web = Chrome + Angular.js + Ionic

Desktop = Electron + Angular.js + Ionic

Mobile = Cordova + Angular.js + Ionic

除了前面的WebView不一样，后面都是Angular.js + Ionic。

步骤
---

###Step 1: 从Web到混合应用，再到桌面应用

在最打开的时候它只是一个单纯的混合应用，我想总结一下我的学习经验，分享一下学习的心得，如：

 - 完整的Web开发,运维,部署,维护介绍   
 - 如何写好代码——重构、测试、模式
 - 遗留代码、遗留系统的形成
 - 不同阶段所需的技能
 - 书籍推荐
 - 技术栈推荐
 - Web应用解决方案
 
接着我用Ionic创建了这个应用，这是一个再普通不过的过程。在这个过程里，我一直使用Chrome在调度我的代码。因为我是Android用户，我有Google Play的账号，便发布了Android版本。这时候遇到了一个问题，我并没有Apple Developer账号(现在在申请ing。。)，而主要的用户对象程序员，这是一群**不土**的土豪。

![iPHONE](./images/iphone.jpg)

偶然间我才想到，我只要上传Web版本的代码就可以暂时性实现这个需求了。接着找了个AWS S3的插件，直接上传到了AWS S3上托管成静态文件服务。

几天前在Github上收到一个issue——关于创造桌面版， 我便想着这也是可能的，我只需要写一个启动脚本和编译脚本即可。

所以，最后我们的流程图就如下所示：

![Growth Arch](./images/growth-arch.png)

除了显示到VR设备上，好像什么也不缺了。并且在我之前的文章《[Oculus + Node.js + Three.js 打造VR世界](https://github.com/phodal/oculus-nodejs-threejs-example)》，也展示了Web在VR世界的可能性。

在这实现期间有几个点可以分享一下：

1. 响应式设计
2. 平台/设备特定代码

### Step 2: 响应式设计

响应式设计可以主要依赖于Media Query，而响应式设计主要要追随的一点是不同的设备不同的显示，如：

![full-platforms.jpg](./images/full-platforms.jpg)

这也意味着，我们需要对不同的设备进行一些处理，如在大的屏幕下，我们需要展示菜单：

![gnu-linux.png](./images/gnu-linux.jpg)

而这可以依赖于Ionic的**expose-aside-when="large"**，而并非所有的情形都是这么简单的。如我最近遇到的问题就是图片缩放的问题，之前的图片针对的都是手机版——经过了一定的缩放。

这时在桌面应用上就会出现问题，就需要限定大小等等。

而这个问题相比于平台特定问题则更容易解决。

### Step 3: 平台特定代码

对于特定平台才有的问题就不是一件容易解决的事，分享一下：

#### 存储

我遇到的第一个问题是**数据存储**的问题。最开始的时候，我只需要开始混合应用。因此我可以用**Preferences**、或者**SQLite**来存储数据。

后来，我扩展到了Web版，我只好用LocalStoarge。于是，我就开始抽象出一个**$storageServices**来做相应的事。接着遇到一系列的问题，我舍弃了原有的方案，直接使用LocalStoarge。

#### 数据分析

为了开发方便，我使用Google Analytics来分析用户的行为——毕竟数据对我来说也不是特别重要，只要可以看到有人使用就可以了。

这时候遇到的一个问题是，我不需要记录Web用户的行为，但是我希望可以看到有这样的请求发出。于是对于Web用户来说，只需要：

```js
        trackView: function (view) {
          console.log(view);
        }
```

而对于手机用户则是:

```js
      trackView: function (view) {
        $window.analytics.startTrackerWithId('UA-71907748-1');
        $window.analytics.trackView(view)
      }
```

这样在我调试的时候我只需要打个Log，在产品环境时就会Track。

#### 自动更新

同样的，对于Android用户来说，他们可以选择自行下载更新，所以我需要针对Android用户有一个自动更新：

```
var isAndroid = ionic.Platform.isAndroid();
if(isAndroid) {
  $updateServices.check('main');
}
```    

#### 桌面应用

对于桌面应用来说也会有类似的问题，我遇到的第一个问题是Electron默认开启了AMD。于是，直接删之：

```html
<script>
//remove module for electron
if(typeof module !== 'undefined' && module && module.exports){
  delete module;
}
</script>
```  

类似的问题还有许多，不过由于应用内容的限制，这些问题就没有那么严重了。

如果有一天，我有钱开放这个应用的应用号，那么我就会再次献上这个图：

![六边形架构](./images/hexoarch.png)

### 未来

我就开始思索这个问题，未来的趋势是合并到一起，而这一个趋势在现在就已经是完成时了。

那么未来呢？你觉得会是怎样的？
