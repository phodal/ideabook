JavaScript制作Slide框架
===

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

由于我是一个懒人，所以在实现基本的Slide功能时，我找到了一个名为``bespoke``的迷你框架。原理大致和大家分享一下，在这个库里一共有下面几个函数：

 - readURL() 读取URL来获取当前的页数，将跳转到相应的页数。
 - activate(index, customData) 主要的函数，实际上就是切换className而已——将新的页面标记为'active'。
 - writeURL(index) 切换slide的时候，更新URL的hash
 - step(offset, customData) 计算页面
 - on(eventName, callback) 事件监听函数
 - fire(eventName, eventData) 事件触发函数
 - createEventData (el, eventData) 创建事件的数据

大致的功能就如上所说的，相当简单。

### Step 2: 解析Markdown

接着，我们就可以创建解析Markdown的功能了，遗憾的是这里的代码我也是用别人的——``micromarkdown``，一个非常简单的Markdown解析器。

### Step 3: 事件处理

在我们完成了基本的Slide功能后，我们就可以处理一些特殊的事件，如移动设备和键盘事件。在EW初始化时，我们可以会trigger一个名为``ew:slide:init``的事件来告诉其他组件系统已经初始化了。这时在我们对应的事件处理函数中，我们就可以判断它是不是移动设备:

```javascript
slides = document.getElementsByTagName('section');
syncSliderEventHandler();

if (slides && isTouchDevice && window.slide) {
	touchDeviceHandler();
}
```

如果是移动设备，我们会额外的监听三个事件：

 - touchstart
 - touchend
 - touchmove

如果是键盘输入的话，那么依据不同的按键做不同的处理：

```javascript
document.addEventListener("keyup", function (event) {
	var keyCode = event.keyCode;
	if (keyCode === TAB || ( keyCode >= SPACE && keyCode <= PAGE_DOWN ) || (keyCode >= LEFT && keyCode <= DOWN)) {
		switch (keyCode) {
			case  PAGE_UP:
			case  LEFT:
			case  UP:
				window.slide.prev();
				break;
			case TAB:
			case SPACE:
			case PAGE_DOWN:
			case  RIGHT:
			case DOWN:
				window.slide.next();
				break;
		}

		event.preventDefault();
	}
});
```

如向上就展示下一张幻灯片，向下就展示下一张幻灯片。

### Step 4: 解析字幕

在EchoesWorks中提供了一个很有趣的功能——类似于听歌时的歌词显示，并且可以自动播放和切换。它并没有用到什么特殊的技能，只是简单的对比时间，并且替换文字。

```javascript
if (that.time < nextTime && words.length > 1) {
		var length = words.length;
		var currentTime = that.parser.parseTime(that.data.times)[currentSlide];
		var time = nextTime - currentTime;
		var average = time / length * 1000;
		var i = 0;
		document.querySelector('words').innerHTML = words[i].word;

		timerWord = setInterval(function () {
			i++;
			if (i - 1 === length) {
				clearInterval(timerWord);
			} else {
				document.querySelector('words').innerHTML = words[i].word;
			}
		}, average);
	}
	return timerWord;
}
```

### Step 5: 进度条

一如既往的，我们的进度条，还是用别人已经写好的组件。这里我们用到的是nanobar。在Nanobar里，我们定义了一个go函数，在这个函数里来转换进度条：

```
Nanobar.prototype.go = function (p) {
	this.bars[0].go(p);
	if (p == 100) {
		init.call(this);
	}
};
```

在我们的slide文件里，再对其进行处理：

```
window.bar.go(100 * ( index + 1) / slides.length);
```


### Step 6: 同步

在这里并没有什么特别高级的用法，只是简单的事件监听

```javascript
function handler() {
	window.slide.slide(parseInt(localStorage.getItem('echoesworks'), 10));
}

if (window.addEventListener) {
	window.addEventListener("storage", handler, false);
} else {
               // IE
	window.attachEvent("onstorage", handler);
}
```

即，当监听到调用``storage``的方法，就会跳转到相应的页面。

正常情况下，我们只用一个标签来展示我们的slide。当我们有另外一个标签的时候，我们就可以存储当前的slide。

```javascript
localStorage.setItem('echoesworks', index);
```

这样就可以实现，在一个页面到下一页时，另外一个标签也会跳到下一页。

### 练习建议
  