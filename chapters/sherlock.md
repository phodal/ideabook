###从零开始设计技能树: 使用Graphviz建立模型

在开始设计新的技能树——[Sherlock](https://github.com/phodal/sherlock)的同时，结合一下原有的技能树，说说如何去设计，新的技能树还很丑。

![Sherlock][1]

##Graphviz

>  Graphviz （英文：Graph Visualization Software的缩写）是一个由AT&T实验室启动的开源工具包，用于绘制DOT语言脚本描述的图形。它也提供了供其它软件使用的库。Graphviz是一个自由软件，其授权为Eclipse Public License。其Mac版本曾经获得2004年的苹果设计奖。

一个简单的示例代码如下:

	graph example1 {
	    Server1 -- Server2
	    Server2 -- Server3
	    Server3 -- Server1
	}

执行编译后:

    dot -Tjpg lz.dot -o lz.jpg

就会生成下面的图片

![lz][2]
	
接着我们便可以建立一个简单的模型来构建我们的技能树。

##简单的技能树

先以JavaScript全栈作一个简单的示例，他们可能存在下面的依赖关系:


 - "JavaScript" -> "Web前端"
 - "HTML" -> "Web前端"
 - "CSS" -> "Web前端"
 - "Web前端" -> "Web开发"
 - "JavaScript" -> "Node.js" -> "Web服务端"
 - "SQL/NoSQL" -> "Web服务端"
 - "Web Server-Side" -> "Web开发"	

	
即Web前端依赖于JavaScript、HTML、CSS，而Node.js依赖于JavaScript，当然我们也需要数据的支持，大部分的网站都是数据驱动型的开发。而构成完成的开发链的则是前端 + 服务端。

于是我们有了这张图:

![Tree][3]

而我们的代码是这样的:

```c
    digraph tree
    {
        nodesep=0.5;
        charset="UTF-8";
        rankdir=LR;
        fixedsize=true;
        node [style="rounded,filled", width=0, height=0, shape=box, fillcolor="#E5E5E5", concentrate=true]
        "JavaScript" ->"Web前端"
        "HTML" -> "Web前端"
        "CSS" -> "Web前端"
        "Web前端" -> "Web开发"
        "JavaScript" -> "Node.js" -> "Web服务端"
        "SQL/NoSQL" -> "Web服务端"
        "Web服务端" -> "Web开发"
    }
```
    
上面举出的是一个简单的例子，对应的我们可以做一些更有意思的东西，比如将dot放到Web上，详情见下一篇。


  [1]: /static/media/uploads/sherlock.png
  [2]: /static/media/uploads/lz.jpg
  [3]: /static/media/uploads/tree.jpg

#技能树之旅: 计算点数与从这开始

之前写了一篇[技能树之旅: 从模块分离到测试](http://www.phodal.com/blog/rebuild-skilltree-from-module-split-to-test/)，现在来说说这其中发生了什么。

##从这开始

在我们没有点击任何技能的时候，显示的是"从这开始"，而当我们点下去时发生了什么?

![Start](http://www.phodal.com//static/media/uploads/start.jpg)

明显变化如下:

 - 样式变了
 - URL变成了[http://skill.phodal.com/#_a2_1_Name](http://skill.phodal.com/#_a2_1_Name)
 - 点数 + 1
 - 点亮了箭头
 
###从Knockout开始

> Knockout是一个轻量级的UI类库，通过应用MVVM模式使JavaScript前端UI简单化。

据说有下面的一些特性。

 - 声明式绑定 (Declarative Bindings)：使用简明易读的语法很容易地将模型(model)数据关联到DOM元素上。
 - UI界面自动刷新 (Automatic UI Refresh)：当您的模型状态(model state)改变时，您的UI界面将自动更新。
 - 依赖跟踪 (Dependency Tracking)：为转变和联合数据，在你的模型数据之间隐式建立关系。
 - 模板 (Templating)：为您的模型数据快速编写复杂的可嵌套的UI。
 
在我们的html中的从这开始是这样一段HTML

    <h2 class="start-helper" data-bind="css:{active:noPointsSpent}">从这开始!</h2> 

这是对应的CSS:

    .start-helper-avatar {
      background: url(../images/red-arrow.png) no-repeat left center;
      padding-left: 55px;
      top: 80px;
      position: relative;
      left: 410px;
      -moz-opacity: 0;
      -khtml-opacity: 0;
      -webkit-opacity: 0;
      opacity: 0;
      -ms-filter: progid:DXImageTransform.Microsoft.Alpha(opacity=0);
      filter: alpha(opacity=0);
    }
    
    .start-helper-avatar.active {
      -moz-opacity: 1;
      -khtml-opacity: 1;
      -webkit-opacity: 1;
      opacity: 1;
      -ms-filter: progid:DXImageTransform.Microsoft.Alpha(opacity=100);
      filter: alpha(opacity=100);
    }

``.start-helper-avatar.active``与``.start-helper-avatar``的不同之处在于.actie将opacity设置为了1。

而，我们对应的JS代码是这样子的:

    self.noPointsSpent = ko.computed(function () {
      return !Boolean(ko.utils.arrayFirst(self.skills(), function (skill) {
        return (skill.points() > 0);
      }));
    });
    
当有一个技能点数大于0时，返回False。而当没有技能点数时，html是这样的。    

    <h2 class="start-helper active" data-bind="css:{active:noPointsSpent}">从这开始!</h2>
    
故而对于此，我们可以明白，Knockout的CSS绑定是这样子的:

> CSS绑定主要是给DOM元素对象添加或移除一个或多个css class类名。这非常有用，比如当值变成负数的时候用红色高亮显示。

##点数计算

对应的我们可以找到点数计算的HTML

    <div data-bind="css: { 'can-add-points': canAddPoints, 'has-points': hasPoints, 'has-max-points': hasMaxPoints }, attr: { 'data-skill-id': id }" class="skill">
    
当然还有:
    
    <div data-bind="click: addPoint, rightClick: removePoint" class="hit-area"></div>
    

与CSS:

    .skill.can-add-points .frame {
      background-position: -80px top;
    }
    .skill.can-add-points .skill-dependency {
      -moz-opacity: 1;
      -khtml-opacity: 1;
      -webkit-opacity: 1;
      opacity: 1;
      -ms-filter: progid:DXImageTransform.Microsoft.Alpha(opacity=100);
      filter: alpha(opacity=100);
    }

对应的，我们可以找到它的js函数:

      self.addPoint = function () {
        if (self.canAddPoints()) {
          self.points(self.points() + 1);
        }
      };
      self.removePoint = function () {
        if (self.canRemovePoints()) {
          self.points(self.points() - 1);
        }
      };
      
看上去通俗易懂，唯一需要理解的就是``click``。

> click绑定在DOM元素上添加事件句柄以便元素被点击的时候执行定义的JavaScript 函数。

##其他

Sherlock:一个新的技能树:[https://github.com/phodal/sherlock](https://github.com/phodal/sherlock)。

开发进行时，欢迎加入。

  [1]: /static/media/uploads/start.jpg  
  
使用D3.js与Darge-d3构建一个简单的技能树的时候，需要一个简单的类似于小贴士的插件。

![Tooltips][1]

##Tooltipster

Tooltipster是一个jQuery tooltip 插件，兼容Mozilla Firefox, Google Chrome, IE8+。

简单示例``html``:

        <section class="container tooltip" title="Parent container">
		<a href="http://google.com" class="tooltip" title="Get your Google on">Google</a>
	</section>

简单示例``js`:

		$(document).ready(function() {
			$('.tooltip').tooltipster();
		});

###D3.js Tooltipster Require配置

D3.js、Tooltipster与Requirejs的配置如下所示:

	require.config({
	  baseUrl: 'app',
	  paths: {
	    jquery: 'lib/jquery-2.1.3',
	    d3: 'lib/d3.min',
	    text: 'lib/text',
	    'jquery.tooltipster': 'lib/jquery.tooltipster.min'
	  },
	  'shim': {
	    'jquery.tooltipster': {
	      deps: ['jquery']
	    }
	  }
	});

###整合代码

最后代码如下所示:

      inner.selectAll('g.node')
        .each(function (v, id) {


          g.node(v).books = Utils.handleEmptyDocs(g.node(v).books);
          g.node(v).links = Utils.handleEmptyDocs(g.node(v).links);

          var data = {
            id: id,
            name: v,
            description: g.node(v).description,
            books: g.node(v).books,
            links: g.node(v).links
          };
          var results = lettuce.Template.tmpl(description_template, data);

          $(this).tooltipster({
            content: $(results),
            contentAsHTML: true,
            position: 'left',
            animation: 'grow',
            interactive: true});
          $(this).find('rect').css('fill', '#ecf0f1');
        });

##结束

代码见： [https://github.com/phodal/sherlock](https://github.com/phodal/sherlock)

  [1]: /static/media/uploads/tips.jpg  