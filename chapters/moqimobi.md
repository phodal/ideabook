单页面移动应用
===

概况
---

### 背景

看到项目上的移动框架，网上寻找了一下，发现原来这些一开始都有。于是，找了个示例开始构建一个移动平台的CMS——[墨颀 CMS](http://cms.moqi.mobi)，方便项目深入理解的同时，也可以自己维护一个CMS系统。

### Showcase

GitHub: [http://github.com/phodal/moqi.mobi](http://github.com/phodal/moqi.mobi)

Demo: [墨颀 CMS](http://cms.moqi.mobi)

### jQuery + Backbone + UnderScore + Require.JS

尝试过用AngularJS和EmberJS，发现对于使用AngularJS以及EmberJS来说，主要的问题是要使用自己熟悉的东西没那么容易引入。而且考虑到谷歌向来对自己的项目的支持不是很好~~，所以便放弃了AngluarJS的想法。

于是开始寻找一些方案，但是最后还是选择了一个比较通用的方案。

 - RequireJS
 - jQuery
 - Underscore
 - Backbone

相对于AngularJS来说，Backbone是一个轻量级的方案，从大小上来说。对于自己来说，灵活性算是其中好的一点，也就是自己可以随意的加入很多东西。

**关于Backbone**

> Backbone.js是一套JavaScript框架与RESTful JSON的应用程式接口。也是一套大致上符合MVC架构的编程范型。Backbone.js以轻量为特色，只需依赖一套Javascript 函式库即可运行。

具体功能上应该是

 - Backbone 轻量级，支持jquery，自带路由，对象化视图，强大的sync机制减少页面大小从而加快页面显示。
 - jQuery jQuery使用户能更方便地处理HTML（标准通用标记语言下的一个应用）、events、实现动画效果，并且方便地为网站提供AJAX交互。不过主要是jQuery能够使用户的html页面保持代码和html内容分离，只需定义id即可。
 - Underscore是Backbone的依赖库  Underscore 是一个JavaScript实用库,提供了类似Prototype.js的一些功能,但是没有继承任何JavaScript内置对象。
  - RequireJS 你可以顺序读取仅需要相关依赖模块。

前台UI，使用的是Pure CSS，一个轻量级的CSS框架，但是最后感觉，总体用到一起，大小还是相当的。只是可以有一个更好的移动体验。

****其他可替换的框架**

**AngularJS**，考虑到某些因素，可能会替换掉Backbone，但是还不是当前可行的方案。为了学习是一方案，也为了更好的普及某些东西。

**handlebars**  Handlebars 是Mustache的改进，显示与逻辑分离，语法兼容Mustache，可以编译成代码，改进Mustache对路径的支持，但是若需要在服务端运行需要使用服务端Javascript引擎如Node.js。

**项目**

前后端分离设计，后台对前台只提供JSON数据，所以在某种意义上来说可能会只适合浏览，和这个要配合后台的框架。总的来说，适合于阅读类的网站。

**源码**

代码依然是放在Github上，基本功能已经可以Works了。

[https://github.com/gmszone/moqi.mobi](https://github.com/gmszone/moqi.mobi)

步骤
---

### Step 1: 使用Require.js管理依赖

**库及依赖**

这里用的是bower的JS来下载库，详细可以参考一下[bower install js使用bower管理js](http://www.phodal.com/blog/use-bower-to-install-js-plugins/) 这篇文章。

需要下载的库有

 - RequireJS
 - Backbone
 - Underscore
 - Mustache
 - jQuery

引用官网的示例

	<!DOCTYPE html>
	<html>
	    <head>
	        <title>My Sample Project</title>
	        <!-- data-main attribute tells require.js to load
	             scripts/main.js after require.js loads. -->
	        <script data-main="js/main" src="lib/require.js"></script>
	    </head>
	    <body>
	        <h1>My Sample Project</h1>
	    </body>
	</html>

我们需要一个require.js和一个main.js放在同一个目录，在main.js中用使用require()来载入需要加载的脚本。

	require.config({
	    baseUrl: 'lib/',
	    paths: {
	        jquery: 'jquery-2.1.1.min'
	    },
	    shim: {
	        underscore: {
	            exports: '_'
	        }
	    }
	});

	require(['../app'], function(App){
	    App.initialize();
	});

在config中可以配置好其他的库，接着调用了app.js。

	define(['jquery', 'underscore'], function($, _){
	    var initialize = function() {
	        console.log("Hello World");
	    }

	    return {
	        initialize: initialize
	    };
	});

当打开index.html的时候便会在console中输出``Hello World``。这样我们就完成一个基本的框架，只是还没有HTML，
文件列表如下所示

	.
	|____app.js
	|____backbone.js
	|____HomeView.js
	|____index.html
	|____jquery.js
	|____main.js
	|____mustache.js
	|____require.js
	|____router.js
	|____text.js
	|____underscore.js
	
在这里有些混乱，但是为了少去其中的一些配置的麻烦，就先这样讲述。

### Step 2: 添加路由

用Backbone的一个目的就在于其的路由功能，于是便添加这样一个js——``router.js``,内容如下所示:

    define([
        'jquery',
        'underscore',
        'backbone',
        'HomeView.js'
    ], function($, _, Backbone, HomeView) {

        var AppRouter = Backbone.Router.extend({
            routes: {
                'index': 'homePage',
                '*actions': 'homePage'
            }
        });
        var initialize = function() {
            var app_router = new AppRouter;

            app_router.on('route:homePage', function() {
                var homeView = new HomeView();
                homeView.render();
            });

            Backbone.history.start();
        };
        return {
            initialize: initialize
        };
    });
    
在这里我们先忽略掉HomeView.js，因为这是下面要讲的，在router.js中，我们定义了一个AppRouter，

 - ``index``指向的是在初始化时候定义的homePage，这样就可以将主页转向HomeView.js。
 - ``*actions``便是将其他未匹配的都转向homePage。
 
接着我们需要修改一下``app.js``，让他一运行地时候便可以进入路由选择

	define(['jquery', 'underscore', 'router'], function($, _, Router) {
	    var initialize = function() {
	        Router.initialize();
	    };

	    return {
	        initialize: initialize
	    };
	});
	
也就是初始化一下路由。

### Step 3: 创建主页View

使用Mustache的优点在于，后台仅仅只需要提供数据，并在前台提供一个位置。因此我们修改了下HTML

	<!DOCTYPE html>
	<html>
	    <head>
	        <title>My Sample Project</title>
	        <script data-main="main" src="require.js"></script>
	    </head>
	    <body>
	    	<div id="aboutArea">{{project}}</div>
	    </body>
	</html
	
创建了aboutArea这样一个ID，于是我们便可以很愉快地在HomeView.js中添加project的数据。	

	define([
	    'jquery',
	    'underscore',
	    'mustache',
	    'text!/index.html'
	], function($, _, Mustache, indexTemplate) {

	    var HomeView = Backbone.View.extend({
	        el: $('#aboutArea'),

	        render: function() {
	            var data = {
	                project: "My Sample Project"
	            };
	            this.$el.html(Mustache.to_html(indexTemplate, data));
	        }
	    });

	    return HomeView;
	});


在HomeView.js中，定义了data这样一个object，代码最终的效果便是用"My Sample Project"替换到HTML中的{{project}}。

这样我们便完成了一个真正意义上的移动web CMS平台的Hello,World，剩下的便是添加一个又一个的脚手架。


当看到[墨颀 CMS](http://cms.moqi.mobi/)的菜单，变成一个工具栏的时候，变觉得这一切有了意义。于是就继续看看这样一个CMS的边栏是怎么组成的。

**RequireJS与jQuery 插件示例**

一个简单的组合示例如下所示，在main.js中添加下面的内容

	requirejs.config( {
	    "shim": {
	        "jquery-cookie"  : ["jquery"]
	    }
	} );

接着在另外的文件中添加

	define(["jquery"],
	       function($){
               //添加函数
	});

这样我们就可以完成一个简单的插件的添加。

### Step 4: jQuery Sidr

> The best jQuery plugin for creating side menus and the easiest way for doing your menu responsive

这是一个创建响应式侧边栏的最好的也是最简单的工具，于是我们需要下载jQuery.sidr.min.js到目录中，接着修改一下main.js:

	require.config({
	    baseUrl: 'lib/',
	    paths: {
	        'text': 'text',
	        jquery: 'jquery-2.1.1.min',
	        async: 'require/async',
	        json: 'require/json',
	        mdown: 'require/mdown',
	        router: '../router',
	        templates: '../templates',
	        jquerySidr: 'jquery.sidr.min',
	        markdownConverter : 'require/Markdown.Converter'
	    },
	    shim: {
	        jquerySidr:["jquery"],
	        underscore: {
	            exports: '_'
	        }
	    }
	});

	require(['../app'], function(App){
	    App.initialize();
	});

添加jquery.sidr.min到里面。

引用官方的示例代码

	$(document).ready(function() {
	  $('#simple-menu').sidr();
	});

我们需要将上面的初始化代码添加到app.js的初始化中，

	define([
	    'jquery',
	    'underscore',
	    'backbone',
	    'router',
	    'jquerySidr'
	], function($, _, Backbone, Router){

	    var initialize = function(){
	        $(document).ready(function() {
	            $('#menu').sidr();
	        });
	        Router.initialize();
	    };

	    return {
	        initialize: initialize
	    };
	});

这样打开[墨颀 CMS](http://cms.moqi.mobi)便可以看到最后的效果。


正在一步步完善[墨颀 CMS](http://cms.moqi.mobi/)，在暂时不考虑其他新的功能的时候，先和[自己的博客](http://www.phodal.com)整合一下。

### Step 5: Django Tastypie示例

之前用AngluarJS做的全部文章的时候是Tastypie做的API，只是用来生成的是博客的内容。只是打开的速度好快，可以在1秒内打开，献上URL:

[http://www.phodal.com/api/v1/url/?offset=0&limit=20&format=json](http://www.phodal.com/api/v1/url/?offset=0&limit=20&format=json)

之前只是拿Tastypie生成一些简单的JSON数据，如keywords_string,slug,title这些简单的数据。

因为这里的Blogpost是来自mezzanine，原来的``api.py``，如下所示：

    from tastypie.resources import ModelResource
    from mezzanine.blog.models import BlogPost, BlogCategory


    class AllBlogSlugResource(ModelResource):
        class Meta:
            queryset = BlogPost.objects.published()
            resource_name = "url"
            fields = ['keywords_string', 'slug', 'title']
            allowed_methods = ['get']


    class BlogResource(ModelResource):
        class Meta:
            queryset = BlogPost.objects.published()
            resource_name = "blog"
            fields = ['keywords_string', 'slug', 'title', 'content', 'description']
            allowed_methods = ['get']

而这时为了测试方便，还需要解决跨域请求的问题，生成的内容大致如下所示:


   	{
	  "meta": {
	    "limit": 1,
	    "next": "/api/v1/url/?offset=1&limit=1&format=json",
	    "offset": 0,
	    "previous": null,
	    "total_count": 290
	  },
	  "objects": [
	    {
	      "keywords_string": "jquery backbone mustache underscore siderbar",
	      "resource_uri": "/api/v1/url/369/",
	      "slug": "use-jquery-backbone-mustache-build-mobile-app-cms-add-jquery-plugins",
	      "title": "构建基于Javascript的移动web CMS——添加jQuery插件"
	    }
	  ]
	}

#### 跨域支持

于是网上搜索了一下，有了下面的代码:

    from tastypie.resources import Resource, ModelResource
    from mezzanine.blog.models import BlogPost, BlogCategory
    from django.http.response import HttpResponse
    from tastypie.exceptions import ImmediateHttpResponse
    from tastypie import http
    from tastypie.serializers import Serializer

    class BaseCorsResource(Resource):

        def create_response(self, *args, **kwargs):
            response = super(BaseCorsResource, self).create_response(*args, **kwargs)
            response['Access-Control-Allow-Origin'] = '*'
            response['Access-Control-Allow-Headers'] = 'Content-Type'
            return response

        def post_list(self, request, **kwargs):

            response = super(BaseCorsResource, self).post_list(request, **kwargs)
            response['Access-Control-Allow-Origin'] = '*'
            response['Access-Control-Expose-Headers'] = 'Location'
            return response
        
        def method_check(self, request, allowed=None):

            if allowed is None:
                allowed = []
     
            request_method = request.method.lower()
            allows = ','.join(map(lambda s: s.upper(), allowed))
     
            if request_method == 'options':
                response = HttpResponse(allows)
                response['Access-Control-Allow-Origin'] = '*'
                response['Access-Control-Allow-Headers'] = 'Content-Type'
                response['Access-Control-Allow-Methods'] = "GET, PUT, POST, PATCH"
                response['Allow'] = allows
                raise ImmediateHttpResponse(response=response)
     
            if not request_method in allowed:
                response = http.HttpMethodNotAllowed(allows)
                response['Allow'] = allows
                raise ImmediateHttpResponse(response=response)
     
            return request_method

    class AllBlogSlugResource(BaseCorsResource, ModelResource):
        class Meta:
            queryset = BlogPost.objects.published()
            resource_name = "url"
            fields = ['keywords_string', 'slug', 'title']
            allowed_methods = ['get']
            serializer = Serializer()

    class BlogResource(BaseCorsResource, ModelResource):
        class Meta:
            queryset = BlogPost.objects.published()
            resource_name = "blog"
            fields = ['keywords_string', 'slug', 'title', 'content', 'description']
            allowed_methods = ['get']
            serializer = Serializer()

接着便可以很愉快地、危险地跨域。

#### 整合

接着修改了一下代码中configure.json的blogListUrl，以及模块

        <div class="l-box blogPosts">
            <h2>动 态</h2>
            {{#objects}}
            <p>
            <!--<span class="date">{{created}}</span>-->
            <a href="#/blog/{{slug}}" alt="{{title}}">{{title}}</a>
            </p>
            {{/objects}}
        </div>

便可以请求到结果了。

一开始对于可配置的选择是正确的.

### Step 6: RequireJS Plugins

网上搜索到一个叫RequireJS Plugins的repo。

里面有这样的几个插件:

 - **async** : Useful for JSONP and asynchronous dependencies (e.g. Google Maps).
 - **font** : Load web fonts using the [WebFont Loader API](https://code.google.com/apis/webfonts/docs/webfont_loader.html)
   (requires `propertyParser`)
 - **goog** : Load [Google APIs](http://code.google.com/apis/loader/)
   asynchronously (requires `async!` plugin and `propertyParser`).
 - **image** : Load image files as dependencies. Option to "cache bust".
 - **json** : Load JSON files and parses the result. (Requires `text!` plugin).
 - **mdown** : Load Markdown files and parses into HTML. (Requires `text!`
   plugin and a markdown converter).
 - **noext** : Load scripts without appending ".js" extension, useful for
   dynamic scripts.

于是，我们可以用到这里的json用来加载JSON文件，虽然也可以用Requirejs的text插件，但是这里的json有对此稍稍的优化。

在后面的部分中我们也用到了mdown，用于显示一个md文件，用法上大致是一样的。

将json.js插件放到目录里，再配置好main.js。

    require.config({
        paths: {
            'text': 'text',
            jquery: 'jquery',
            json: 'require/json'
        },
        shim: {
            underscore: {
                exports: '_'
            }
        }
    });

    require(['app'], function(App) {
        App.initialize();
    });
    

于是我们将HomeView.js中的data变为configure的数据，这样便可以直接使用这个json文件。

    define([
        'jquery',
        'underscore',
        'mustache',
        'text!/index.html',
        'json!/configure.json'
    ], function($, _, Mustache, indexTemplate, configure) {

        var HomeView = Backbone.View.extend({
            el: $('#aboutArea'),

            render: function() {
                this.$el.html(Mustache.to_html(indexTemplate, configure));
            }
        });

        return HomeView;
    });
    
 configure.json的代码如下所示:
 
 	{
	    "project": "My Sample Project"
	}

最后实现的效果和模板结束是一样的，只会在页面上显示

     My Sample Project

在[墨颀 CMS](http://cms.moqi.mobi)中的动态的文章是从我博客的API加载过来的，因为当前没有其他好的CMS当接口。之前直接拿博客的DB文件+Nodejs+RESTify生成了一个博客的API，而且可以支持跨域请求。


### Step 6: 简单的博客

这次我们可以简单的做一个可以供移动平台阅读的博客，除了不能写作以外(ps:不能写作还能叫博客么)。对于写博客的人来说更多的只是写，而对于读者来说，他们只需要读，所以在某种意义上可以将博客的写和读分离开来。

对于用户来说，博客是由两个页面构建的:

 - 博文列表(blogposts list)
 - 博客内容(blogposts detail)

在这里我们先关注博文列表 
 
**博文列表**

博文列表的内容一般有:

 - 作者(Author)
 - 标题(title)
 - 创建时间/修改时间(Time)
 - 关键词(Keywords)
 - 摘要(Description)
 - 链接(Slug)

一个简单的示例如下，也就是我们接下来要用到的**1.json**中的一部分。

    [{
        "title": "构建基于Javascript的移动web CMS入门——简介",
        "slug": "use-jquery-backbone-mustache-build-mobile-app-cms",
        "description": "看到项目上的移动框架，网上寻找了一下，发现原来这些一开始都有。于是，找了个示例开始构建一个移动平台的CMS——墨颀 CMS，方便项目深入理解的同时，也可以自己维护一个CMS系统。",
        "keywords": [
            "backbone",
            "jquery",
            "underscore",
            "mustache"
        ],
        "created": "2014-07-17 14:16:18.035763"
    }]

这里基本上也就有了上面的要素，除了作者，当然因为作者只有一个，所以在里面写作者就是在浪费流量和钱啊。接着我们就是要把上面的内容读取出来放到CMS里。和之前不同的是，虽然我们可以用和[墨颀CMS文件 JSON文件](http://www.phodal.com/blog/use-jquery-backbone-mustache-build-mobile-app-cms-json-configure/)一样的方法，但是显然这种方法很快就会不适用。


#### 获取在线数据

这里会用到Backbone的Model，我们先创建一个Model

    var BlogPostModel = Backbone.Model.extend({
        name: 'Blog Posts',
        url: function(){
            return this.instanceUrl;
        },
        initialize: function(props){
            this.instanceUrl = props;
        }
    });
    
我们需要在初始化的时候传入一个URL，以便在``getBlog``的时候用到，为了方便调试将url改为同路径的1.json文件

        getBlog: function() {
            var url = '/1.json';
            var that = this;
            collection = new BlogPostModel;
            collection.initialize(url);
            collection.fetch({
                success: function(collection, response){
                    that.render(response);
                }
            });
        },
        
这样当成功获取数据的时候便render页面。最后的HomeView.js代码如下所示:

    define([
        'jquery',
        'underscore',
        'mustache',
        'text!/index.html',
        'text!/blog.html'
    ], function($, _, Mustache, indexTemplate, blogTemplate) {

        var BlogPostModel = Backbone.Model.extend({
            name: 'Blog Posts',
            url: function(){
                return this.instanceUrl;
            },
            initialize: function(props){
                this.instanceUrl = props;
            }
        });

        var HomeView = Backbone.View.extend({
            el: $('#aboutArea'),

            initialize: function(){
                this.getBlog();
            },

            getBlog: function() {
                var url = '/1.json';
                var that = this;
                collection = new BlogPostModel;
                collection.initialize(url);
                collection.fetch({
                    success: function(collection, response){
                        that.render(response);
                    }
                });
            },

            render: function(response) {
                this.$el.html(Mustache.to_html(blogTemplate, response));
            }
        });

        return HomeView;
    });
   
这样也就意味着我们需要在index.html中创建一个id为aboutArea的div。接着我们需要创建一个新的Template——blog.html，它的内容就比较简单了，只是简单的Mustache的使用。

	{{#.}}
	<h2><a href="{{slug}}" alt="{{title}}">{{title}}</a></h1>
	<p>{{description}}</p>
	{{/.}}
	
``{{#.}}``及``{{/.}}``可以用于JSON数组，即循环，也可以是判断是否存在。

最后的结果便是:

    <h2><a href="use-jquery-backbone-mustache-build-mobile-app-cms" alt="构建基于Javascript的移动web CMS入门——简介">构建基于Javascript的移动web CMS入门——简介</a></h2>
    <p>看到项目上的移动框架，网上寻找了一下，发现原来这些一开始都有。于是，找了个示例开始构建一个移动平台的CMS——墨颀 CMS，方便项目深入理解的同时，也可以自己维护一个CMS系统。</p>
    
把description去掉，再修改一个CSS，便是我们在首页看到的结果。

下一次我们将打开这些URL。    

#### 如何查看是否支持JSON跨域请求

本次代码下载:[https://github.com/gmszone/moqi.mobi/archive/0.1.1.zip](https://github.com/gmszone/moqi.mobi/archive/0.1.1.zip)

一个简单的工具就是

    curl I -s http://example.com
    
在这里我们查看

    curl -I -s http://api.phodal.net/blog/page/1

应该要返回	Access-Control-Allow-Origin: *


	HTTP/1.1 200 OK
	Server: mokcy/0.17.0
	Date: Thu, 24 Jul 2014 00:38:19 GMT
	Content-Type: application/json; charset=utf-8
	Content-Length: 3943
	Connection: keep-alive
	Vary: Accept-Encoding
	Access-Control-Allow-Origin: *
	Access-Control-Allow-Headers: X-Requested-With
	Cache-Control: max-age=600
    
在有了上部分的基础之后，我们就可以生成一个博客的内容——BlogPosts Detail。这样就完成了我们这个[移动CMS](http://cms.moqi.mobi)的几乎主要的功能了，有了上节想必对于我们来说要获取一个文章已经不是一件难的事情了。

#### 获取每篇博客

于是我们照猫画虎地写了一个``BlogDetail.js``

    define([
        'jquery',
        'underscore',
        'mustache',
        'text!/blog_details.html'
    ],function($, _, Mustache, blogDetailsTemplate){

        var BlogPostModel = Backbone.Model.extend({
            name: 'Blog Posts',
            url: function(){
                return this.instanceUrl;
            },
            initialize: function(props){
                this.instanceUrl = props;
            }
        });

        var BlogDetailView = Backbone.View.extend ({
            el: $("#content"),

            initialize: function () {
            },

            getBlog: function(slug) {
                url = "http://api.phodal.net/blog/" + slug;
                var that = this;
                collection = new BlogPostModel;
                collection.initialize(url);
                collection.fetch({
                    success: function(collection, response){
                        that.render(response);
                    }
                });
            },

            render: function(response){
                this.$el.html(Mustache.to_html(blogDetailsTemplate, response));
            }
        });

        return BlogDetailView;
    });

又写了一个``blog_details.html``，然后，然后

    <div class="information pure-g">
        {{#.}}
        <div class="pure-u-1 ">
            <div class="l-box">
                <h3 class="information-head"><a href="#/blog/{{slug}}" alt="{{title}}">{{title}}</a></h3>
                <p>
                    发布时间:<span>{{created}}</span>
                <p>
                    {{{content}}}
                </p>

                </p>
            </div>
        </div>
        {{/.}}
    </div>
    
我们显然需要稍微地修改一下之前``blog.html``的模板，为了让他可以在前台跳转

    {{#.}}
    <h2><a href="#/{{slug}}" alt="{{title}}">{{title}}</a></h2>
    <p>{{description}}</p>
    {{/.}}
    
问题出现了，我们怎样才能进入最后的页面？

#### 添加博文的路由

在上一篇结束之后，每个博文都有对应的URL，即有对应的slug。而我们的博客的获取就是根据这个URL，获取的，换句话说，这些事情都是由API在做的。这里所要做的便是，获取博客的内容，再render。这其中又有一个问题是ajax执行的数据无法从外部取出，于是就有了上面的getBlog()调用render的方法。

我们需要传进一个参数，以便告诉BlogDetail需要获取哪一篇博文。

        routes: {
            'index': 'homePage',
            'blog/*slug': 'blog',
            '*actions': 'homePage'
        }

``*slug``便是这里的参数的内容，接着我们需要调用getBlog(slug)对其进行处理。

        app_router.on('route:blog', function(blogSlug){
            var blogDetailsView = new BlogDetail();
            blogDetailsView.getBlog(blogSlug);
        });
        
最后，我们的``router.js``的内容如下所示:

    define([
        'jquery',
        'underscore',
        'backbone',
        'HomeView',
        'BlogDetail'
    ], function($, _, Backbone, HomeView, BlogDetail) {

        var AppRouter = Backbone.Router.extend({
            routes: {
                'index': 'homePage',
                'blog/*slug': 'blog',
                '*actions': 'homePage'
            }
        });
        var initialize = function() {
            var app_router = new AppRouter;

            app_router.on('route:homePage', function() {
                var homeView = new HomeView();
                homeView.render();
            });

            app_router.on('route:blog', function(blogSlug){
                var blogDetailsView = new BlogDetail();
                blogDetailsView.getBlog(blogSlug);
            });

            Backbone.history.start();
        };
        return {
            initialize: initialize
        };
    });        
    
接着我们便可以很愉快地打开每一篇博客查看里面的内容了。

当前[墨颀CMS](http://cms.moqi.mobi/)的一些基础功能设计已经接近尾声了，在完成博客的前两部分之后，我们需要对此进行一个简单的重构。为的是提取出其中的获取Blog内容的逻辑，于是经过一番努力之后，终于有了点小成果。

### Step 7: 重构

我们想要的结果，便是可以直接初始化及渲染，即如下的结果:

        initialize: function(){
            this.getBlog();
        },

        render: function(response){
            var about = {
                about:aboutCMS,
                aboutcompany:urlConfig["aboutcompany"]
            };
            response.push(about);
            this.$el.html(Mustache.to_html(blogPostsTemplate, response));
        }

为的便是简化其中的逻辑，将与View无关的部分提取出来，最后的结果便是都放在初始化里，显然我们需要一个``render``，只是暂时放在``initialize``应该就够了。下面便是最后的结果:

        initialize: function(){
            var params='#content';
            var about = {
                about:aboutCMS,
                aboutcompany:configure["aboutcompany"]
            };
            var blogView = new RenderBlog(params, '/1.json', blogPostsTemplate);
            blogView.renderBlog(about);
        }

我们只需要将id、url、template传进去，便可以返回结果，再用getBlog部分传进参数。再渲染结果，这样我们就可以提取出两个不同View里面的相同的部分。

#### 构建函数

于是，我们就需要构建一个函数RenderBlog，只需要将id,url,template等传进去就可以了。

    var RenderBlog = function (params, url, template) {
        this.params = params;
        this.url = url;
        this.template = template;
    };

用Javascript的原型继承就可以实现这样的功能，虽然还不是很熟练，但是还是勉强用了上来。

    RenderBlog.prototype.renderBlog = function(addInfo) {
        var template = this.template;
        var params = this.params;
        var url = this.url;
        var collection = new BlogPostModel;

        collection.initialize(url);
        collection.fetch({
            success: function(collection, response){
                if(addInfo !== undefined){
                    response.push(addInfo);
                }
                RenderBlog.prototype.render(params, template, response);
            }
        });
    };

    RenderBlog.prototype.render = function(params, template, response) {
        $(params).html(Mustache.to_html(template, response));
    };

大致便是将原来的函数中的功能抽取出来，再调用自己的方法。于是就这样可以继续进行下一步了，只是暂时没有一个明确的方向。

在和几个有兴趣做**移动CMS**的小伙伴讨论了一番之后，我们觉得当前比较重要的便是统一一下RESTful API。然而最近持续断网中，又遭遇了一次停电，暂停了对API的思考。在周末无聊的时光了看了《人间失格》，又看了会《一个人流浪，不必去远方》。开始思考所谓的技术以外的事情，或许这将是下一篇讨论的话题。

正在我对这个[移动CMS](http://cms.moqi.mobi/)的功能一筹莫展的时候，帮小伙伴在做一个图片滑动的时候，便想着将这个功能加进去，很顺利地找到了一个库。

### Step 8: 移动CMS滑动

我们所需要的两个功能很简单

 - 当用户向右滑动的时候，菜单应该展开
 - 当用户向左滑动的时候，菜单应该关闭
 
在官网看到了一个简单的示例，然而并不是用于这个菜单，等到我完成之后我才知道：为什么不用于菜单？

找到了这样一个符合功能的库，虽然知道要写这个功能也不难。相比于自己写这个库，还不如用别人维护了一些时候的库来得简单、稳定。

> jQuery Plugin to obtain touch gestures from iPhone, iPod Touch and iPad, should also work with Android mobile phones (not tested yet!)
 
然而，它并不会其他一些设备上工作。

**添加jQuery Touchwipe**

添加到requirejs的配置中:

    require.config({
        baseUrl: 'lib/',
        paths: {
            jquery: 'jquery-2.1.1.min',
            router: '../router',
            touchwipe: 'jquery.touchwipe.min'
        },
        shim: {
            touchwipe: ["jquery"],
            underscore: {
                exports: '_'
            }
        }
    });

    require(['../app'], function(App){
        App.initialize();
    });
    
(注：上面的代码中暂时去掉了一部分无关本文的，为了简单描述。)    

接着，添加下面的代码添加到app.js的初始化方法中

        $(window).touchwipe({
            wipeLeft: function() {
                $.sidr('close');
            },
            wipeRight: function() {
                $.sidr('open');
            },
            preventDefaultEvents: false
        });
        
就变成了我们需要的代码。。

    define([
        'jquery',
        'underscore',
        'backbone',
        'router',
        'jquerySidr',
        'touchwipe'
    ], function($, _, Backbone, Router){

        var initialize = function(){
            $(window).touchwipe({
                wipeLeft: function() {
                    $.sidr('close');
                },
                wipeRight: function() {
                    $.sidr('open');
                },
                preventDefaultEvents: false
            });
            $(document).ready(function() {
                $('#sidr').show();
                $('#menu').sidr();
                $("#sidr li a" ).bind('touchstart click', function() {
                    if(null != Backbone.history.fragment){
                        _.each($("#sidr li"),function(li){
                            $(li).removeClass()
                        });

                        $('a[href$="#/'+Backbone.history.fragment+'"]').parent().addClass("active");
                        $.sidr('close');
                        window.scrollTo(0,0);
                    }
                });
            });
            Router.initialize();
        };

        return {
            initialize: initialize
        };
    });

便可以实现我们需要的        

 - 当用户向右滑动的时候，菜单应该展开
 - 当用户向左滑动的时候，菜单应该关闭
 