JavaScript打造Slide应用
===================

概况
---

### 背景

又开始造一个新的轮子了，不过这次的起因比较简单，是想重新发明一个更好的Slide框架 —— EchoesWorks。如名字所言，我所需要的是一个``回声``工坊，即将博客、Slide重新回放。

###Showcase

![EchoesWorks](./images/echoesworks.jpg)

GitHub代码： [https://github.com/phodal/echoesworks](https://github.com/phodal/echoesworks)

### 需求

当前我们有不同的方式可以记录我们的想法、博客、过程，如视频、音频、博客、幻灯片等等。

然而这些并非那么完美，让我们说说这些方式的一些缺陷吧。

1. 视频。有很多技术视频从开始到结束，只有PPT，然后我们就为了这张PPT和声音下了几百M的视频。即使在今天网速很快，但是这并不代表我们可以在我们的手机上放下很多的视频。

2. 音频。音频所受到的限制我想大家都很清楚。什么也不知道~~，什么也看不到，只能听。

3. 博客。博客的主要缺点可能就是不够直接，有时会有点啰嗦。

4. 幻灯片。一个好的PPT，也就意味着上面的内容是很少的。即如果没有人说的时候，就缺少真正有用的东西。

5. 代码。我们真的需要在另外打开一个网址来看代码么?

于是，``EchoesWorks``出现了，它可以支持下面的一些功能：

 - 支持 Markdown
 - Github代码显示
 - 全屏背景图片
 - 左/右侧图片支持
 - 进度条
 - 自动播放
 - 字幕
 - 分屏控制

步骤
---

### Step 1: 基本的Slide功能

### Step 2: 解析Markdown

### Step 3: 事件处理

### Step 4: 解析时间

### Step 5: 进度条

### Step 6: 同步

在这里并没有什么特别高级的用法，只是简单的事件监听

		function handler() {
			window.slide.slide(parseInt(localStorage.getItem('echoesworks'), 10));
		}

		if (window.addEventListener) {
			window.addEventListener("storage", handler, false);
		} else {
                       // IE
			window.attachEvent("onstorage", handler);
		}

即，当监听到调用``storage``的方法，就会跳转到相应的页面。

正常情况下，我们只用一个标签来展示我们的slide。当我们有另外一个标签的时候，我们就可以存储当前的slide。

    localStorage.setItem('echoesworks', index);

这样就可以实现，在一个页面到下一页时，另外一个标签也会跳到下一页。

### 练习建议
  