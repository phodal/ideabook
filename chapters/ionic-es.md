Ionic与ElasticSearch打造O2O应用
===

概况
---

### 背景

搜索引擎是个好东西，GIS也是个好东西。当前还有Django和Ionic。

### Showcase

最后效果图

![elasticsearch_ionic_map](./images/elasticsearch_ionic_map.jpg) 

![elasticsearch_ionic_info_page](./images/elasticsearch_ionic_info_page.jpg)

### 构架设计

对我们的需求进行简要的思考后，设计出了下面的一些简单的架构。

![Django ElasticSearch Ionic 架构](./images/struct.png)

#### 服务端

简单说明:

- 用户在前台或者后台创建数据。
- 在model保存数据的时候，会调用Google的API解析GPS
- 在haystack的配置中设置实时更新，当数据创建的时候自动更新索引
- 数据被ElasticSearch索引

下面是框架的一些简单的介绍

**Django**

> [Django](http://www.phodal.com/blog/tag/django/) 是一个开放源代码的Web应用框架，由Python写成。采用了MVC的软件设计模式，即模型M，视图V和控制器C。它最初是被开发来用于管理劳伦斯出版集团旗下的一些以新闻内容为主的网站的。并于2005年7月在BSD许可证下发布。这套框架是以比利时的吉普赛爵士吉他手Django Reinhardt来命名的。

> [Django](http://www.phodal.com/blog/tag/django/) 的主要目标是使得开发复杂的、数据库驱动的网站变得简单。Django注重组件的重用性和“可插拔性”，敏捷开发和DRY法则（Don't Repeat Yourself）。在Django中Python被普遍使用，甚至包括配置文件和数据模型。


首先考虑Django，而不是其他Node或者Ruby框架的原因是：

- 内置认证系统
- 内置CSRF

当然这是其他框架也所拥有的，主要特性还有:

- 一个表单序列化及验证系统，用于HTML表单和适于数据库存储的数据之间的转换。
- 一套协助创建地理信息系统（GIS）的基础框架

最后一个才是亮点，内置GIS，虽然没怎么用到，但是至少在部署上还是比较方便的。

**Haystack**

> Haystack provides modular search for Django. It features a unified, familiar API that allows you to plug in different search backends (such as Solr, Elasticsearch, Whoosh, Xapian, etc.) without having to modify your code.

Haystack是为Django提供一个搜索模块blabla..，他的主要特性是可以

>  write your search code once and choose the search engine you want it to run on

也就是说你只需要写你的代码选择你的搜索引擎就可以工作了。

**ElasticSearch**

在上面的Haystack提供了这些一堆的搜索引擎，当然支持地点搜索的只有``Solr``和``ElasticSearch``，他们支持的空间搜索有:

- within 	 
- dwithin 
- distance		 
- order_by(‘distance’) 	 
- polygon 

在文档上没有写Solr的polygon搜索，但是实际上也是支持的(详细见这篇文章: [google map solr polygon 搜索](http://www.phodal.com/blog/google-map-width-solr-use-polygon-search/，用的地图是谷歌，所以需要先学会访问谷歌)。

至于为什么用的是ElasticSearch，是因为之前用Solr做过。。。

#### 客户端 

**简单说明  —— GET**

1. 当我们访问Map View的时候，会调用HTML5获取用户的位置
2. 根据用户的位置定位，设置缩放
3. 根据用户的位置发出ElasticSearch请求，返回结果中带上距离
4. 显示

**简单说明  —— POST**

1. 用户填写数据会发给Django API，并验证
2. 成功时，存入数据库，更新索引。

**Ionic**

> Ionic提供了一个免费且开源的移动优化HTML，CSS和JS组件库，来构建高交互性应用。基于Sass构建和AngularJS 优化。

用到的主要是AngularJS，之前用他写过三个APP。

**Django REST Framework**

与Django Tastypie相比，DRF的主要优势在于Web界面的调试。

步骤
---

### Step 1: Django GIS 设置

1.创建虚拟环境

```bash
virtualenv -p /usr/bin/python2.67 django-elasticsearch
```

2.创建项目

为了方便，这里用的是Mezzanine CMS，相比Django的主要优势是，以后扩展方便。但是对于Django也是可以的。

3.安装依赖

这里我的所有依赖有

 - django-haystack
 - Mezzanine==3.1.10
 - djangorestframework
 - pygeocoder
 - elasticsearch

安装

```bash
pip install requirements.txt
```

4.安装ElasticSearch

CentOS

```bash
wget https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-1.4.2.zip
sudo unzip elasticsearch-1.4.2 -d /usr/local/elasticsearch
rm elasticsearch-1.4.2.zip
cd /usr/local/elasticsearch/elasticsearch-1.4.2/
./bin/plugin install elasticsearch/elasticsearch-cloud-aws/2.4.1
curl -XGET http://localhost:9200
```

Mac OS

```bash
brew install elasticsearch
```

5.Django Geo环境搭建

CentOS等GNU/Linux系统: 可以参照[CentOS Django Geo 环境搭建](http://www.phodal.com/blog/install-geo-django-in-centos/)

MacOS: [Mac OS Django Geo 环境搭建](http://www.phodal.com/blog/django-elasticsearch-geo-solution/)

### Step 2: 配置Haystack

**配置Haystack**

```python
HAYSTACK_SIGNAL_PROCESSOR = 'haystack.signals.RealtimeSignalProcessor'

HAYSTACK_CONNECTIONS = {
    'default': {
        'ENGINE': 'haystack.backends.elasticsearch_backend.ElasticsearchSearchEngine',
        'URL': 'http://127.0.0.1:9200/',
        'INDEX_NAME': 'haystack',
    },
}   
```

``HAYSTACK_SIGNAL_PROCESSOR``是为了可以实时处理。
``HAYSTACK_CONNECTIONS`` 则是配置搜索引擎用的。

**配置Django**

在``settings.py``中的``INSTALLED_APPS``添加

```python
"haystack",
"rest_framework",
```

接着

```bash
python manage.py createdb
python manage.py migreate
```

运行

```bash
python manage.py runserver
```

官方有一个简单的文档说明空间搜索—— [Spatial Search](http://django-haystack.readthedocs.org/en/latest/spatial.html)

里面只有``Solr``和``ElasticSearch``是支持的，当然我们也不需要这么复杂的特性。

创建Django app名为nx，目录结构如下

```
.
|______init__.py
|____api.py
|____models.py
|____search_indexes.py
|____templates
| |____search
| | |____indexes
| | | |____nx
| | | | |____note_text.txt
```

api.py是后面要用的。

### Step 3: Django Haystack Model创建

而一般的model没有什么区别，除了修改了save方法

```python
from django.contrib import admin

from django.contrib.gis.geos import Point
from django.core import validators
from django.utils.translation import ugettext_lazy as _
from django.db import models
from pygeocoder import Geocoder

class Note(models.Model):
    title = models.CharField("标题", max_length=30, unique=True)
    latitude = models.FloatField(blank=True)
    longitude = models.FloatField(blank=True)

    def __unicode__(self):
        return self.title

    def save(self, *args, **kwargs):
        results = Geocoder.geocode(self.province + self.city + self.address)
        self.latitude = results[0].coordinates[0]
        self.longitude = results[0].coordinates[1]
        super(Note, self).save(*args, **kwargs)

    def get_location(self):
        return Point(self.longitude, self.latitude)

    def get_location_info(self):
        return self.province + self.city + self.address

admin.site.register(Note)
```

通过``Geocoder.geocode`` 解析用户输入的地址，为了方便直接后台管理了。

### Step 4: 创建search_index

在源码的目录下有一个``search_indexes.py``的文件就是用于索引用的。

```python
from haystack import indexes
from .models import Note

class NoteIndex(indexes.SearchIndex, indexes.Indexable):
    text = indexes.CharField(document=True, use_template=True)
    title = indexes.CharField(model_attr='title')
    location = indexes.LocationField(model_attr='get_location')
    location_info = indexes.CharField(model_attr='get_location_info')

    def get_model(self):
        return Note
```

与些同时我们还需要在``templates/search/indexes/nx/``目录中有``note_text.txt``里面的内容是:

```python
{{ object.title }}
{{ object.get_location }}
{{ object.get_location_info }}
```

**创建数据**

migrate数据库

```bash
python manage.py migrate
```

run

```bash
python manage.py runserver
```

接着我们就可以后台创建数据了。 打开: http://127.0.0.1:8000/admin/nx/note/，把除了``Latitude``和``Longitude``以外的数据都一填——经纬度是自动生成的。就可以创建数据了。

**测试**

访问 http://localhost:9200/haystack/_search

或者

```bash
curl -XGET http://127.0.0.1:9200/haystack/_search
```


如果你没有Ionic的经验，可以参考一下之前的一些文章:[《HTML5打造原生应用——Ionic框架简介与Ionic Hello World》](http://www.phodal.com/blog/ionic-development-android-ios-windows-phone-application/)。

我们用到的库有:

- elasticsearch
- ionic
- ngCordova

将他们添加到``bower.json``，然后执行

```bash
bower install
```

### Step 5: Ionic ElasticSearch 创建页面

1.引入库

在``index.html``中添加

```html
<script src="lib/elasticsearch/elasticsearch.angular.min.js"></script>
<script src="lib/ngCordova/dist/ng-cordova.js"></script>
```

接着开始写我们的搜索模板``tab-search.html``

```html
<ion-view view-title="搜索" ng-controller="SearchCtrl">
    <ion-content>
        <div id="search-bar">
            <div class="item item-input-inset">
                <label class="item-input-wrapper" id="search-input">
                    <i class="icon ion-search placeholder-icon"></i>
                    <input type="search" placeholder="Search" ng-model="query" ng-change="search(query)" autocorrect="off">
                </label>
            </div>
        </div>
    </ion-content>
</ion-view>
```

显示部分

```html
<ion-list>
    <ion-item class="item-remove-animate item-icon-right" ng-repeat="result in results">
        <h2 class="icon-left">{{result.title}}</h2>
        <p>简介: {{result.body}}</p>
        <div class="icon-left ion-ios-home location_info">
            {{result.location_info}}
        </div>
        <div class="button icon-left ion-ios-telephone button-calm button-outline">
            <a ng-href="tel: {{result.phone_number}}">{{result.phone_number}}</a>
        </div>
    </ion-item>
</ion-list>
```

而我们期待的``SearchCtrl``则是这样的

```javascript
$scope.query = "";
var doSearch = ionic.debounce(function(query) {
	ESService.search(query, 0).then(function(results){
		$scope.results = results;
	});
}, 500);

$scope.search = function(query) {
	doSearch(query);
}
```

当我们点下搜索的时候，调用 ESService.

### Step 6: Ionic ElasticSearch Service

接着我们就来构建我们的ESService，下面的部分来自网上:

```javascript
angular.module('starter.services', ['ngCordova', 'elasticsearch'])

.factory('ESService',
  ['$q', 'esFactory', '$location', '$localstorage', function($q, elasticsearch, $location, $localstorage){
    var client = elasticsearch({
      host: $location.host() + ":9200"
    });

    var search = function(term, offset){
      var deferred = $q.defer(), query, sort;
      if(!term){
        query = {
          "match_all": {}
        };
      } else {
        query = {
          match: { title: term }
        }
      }

      var position = $localstorage.get('position');

      if(position){
        sort = [{
          "_geo_distance": {
            "location": position,
            "unit": "km"
          }
        }];
      } else {
        sort = [];
      }

      client.search({
        "index": 'haystack',
        "body": {
          "query": query,
          "sort": sort
        }
      }).then(function(result) {
        var ii = 0, hits_in, hits_out = [];
        hits_in = (result.hits || {}).hits || [];
        for(;ii < hits_in.length; ii++){
          var data = hits_in[ii]._source;
          var distance = {};
          if(hits_in[ii].sort){
            distance = {"distance": parseFloat(hits_in[ii].sort[0]).toFixed(1)}
          }
          angular.extend(data, distance);
          hits_out.push(data);
        }
        deferred.resolve(hits_out);
      }, deferred.reject);

      return deferred.promise;
    };


    return {
      "search": search
    };
  }]
);
```

这个Service主要做的是创建ElasitcSearch Query，然后返回解析结果。


**设计思路**

1. 判断是否有上次记录的位置信息，如果有则将地图的中心设置为上次的位置。
2. 将位置添加到ElasticSearch的Query中。
3. 从ElasticSearch中获取数据，并解析Render到地图上。

**OpenLayer**

> OpenLayers是一个用于开发WebGIS客户端的JavaScript包。OpenLayers 支持的地图来源包括Google Maps、Yahoo、 Map、微软Virtual Earth 等，用户还可以用简单的图片地图作为背景图，与其他的图层在OpenLayers 中进行叠加，在这一方面OpenLayers提供了非常多的选择。除此之外，OpenLayers实现访问地理空间数据的方法都符合行业标准。OpenLayers 支持Open GIS 协会制定的WMS（Web Mapping Service）和WFS（Web Feature Service）等网络服务规范，可以通过远程服务的方式，将以OGC 服务形式发布的地图数据加载到基于浏览器的OpenLayers 客户端中进行显示。OpenLayers采用面向对象方式开发，并使用来自Prototype.js和Rico中的一些组件。

### Step 7: Ionic OpenLayer 地图显示

1.下载OpenLayer

2.添加到``index.html``:

```html
<script src="js/ol.js"></script>
```

**创建NSService**

新建一个``MapCtrl``，需要用到``ESService``和 ``NSService``，NSService是官方示例中的一个函数，提供了一个``getRendererFromQueryString``方法。

```javascript
.factory('NSService', function(){
      var exampleNS = {};

      exampleNS.getRendererFromQueryString = function() {
        var obj = {}, queryString = location.search.slice(1),
            re = /([^&=]+)=([^&]*)/g, m;

        while (m = re.exec(queryString)) {
          obj[decodeURIComponent(m[1])] = decodeURIComponent(m[2]);
        }
        if ('renderers' in obj) {
          return obj['renderers'].split(',');
        } else if ('renderer' in obj) {
          return [obj['renderer']];
        } else {
          return undefined;
        }
      };

      return {
        "exampleNS": exampleNS
      };
})
```

**创建基本地图显示**

这里我们使用的是Bing地图:

```javascirpt
var view = new ol.View({
	center: map_center,
	zoom: 4
});

var controls = ol.control.defaults({rotate: false});
var interactions = ol.interaction.defaults({altShiftDragRotate:false, pinchRotate:false});

var map = new ol.Map({
	controls: controls,
	interactions: interactions,
	layers: [
		new ol.layer.Tile({
			source: new ol.source.BingMaps({
				key: 'Ak-dzM4wZjSqTlzveKz5u0d4IQ4bRzVI309GxmkgSVr1ewS6iPSrOvOKhA-CJlm3',
				culture: 'zh-CN',
				imagerySet: 'Road'
			})
		})
	],
	renderer: NSService.exampleNS.getRendererFromQueryString(),
	target: 'map',
	view: view
});
```

一个简单的地图如上如示。

**获取当前位置**

ngCordova有一个插件是``$cordovaGeolocation``，用于获取当前的位置。代码如下所示:

```javascript
var posOptions = {timeout: 10000, enableHighAccuracy: true};
$cordovaGeolocation
	.getCurrentPosition(posOptions)
	.then(function (position) {
		var pos = new ol.proj.transform([position.coords.longitude, position.coords.latitude], 'EPSG:4326', 'EPSG:3857');

		$localstorage.set('position', [position.coords.latitude, position.coords.longitude].toString());
		$localstorage.set('map_center', pos);

		view.setCenter(pos);
	}, function (err) {
		console.log(err)
	});
```

当获取到位置时，将位置存储到``localstorage``中。

**获取结果并显示**

最后代码如下所示，获取解析后的结果，添加icon

```javascript
ESService.search("", 0).then(function(results){
	var vectorSource = new ol.source.Vector({ });
	$.each(results, function(index, result){
		var position = result.location.split(",");
		var pos = ol.proj.transform([parseFloat(position[1]), parseFloat(position[0])], 'EPSG:4326', 'EPSG:3857');

		var iconFeature = new ol.Feature({
				geometry: new ol.geom.Point(pos),
				name: result.title,
				phone: result.phone_number,
				distance: result.distance
		});
		vectorSource.addFeature(iconFeature);
	});

	var iconStyle = new ol.style.Style({
		image: new ol.style.Icon(({
			anchor: [0.5, 46],
			anchorXUnits: 'fraction',
			anchorYUnits: 'pixels',
			opacity: 0.75,
			src: 'img/icon.png'
		}))
	});

	var vectorLayer = new ol.layer.Vector({
		source: vectorSource,
		style: iconStyle
	});
	map.addLayer(vectorLayer);
});
```

**添加点击事件**

在上面的代码中添加:

```javascript
var element = document.getElementById('popup');

var popup = new ol.Overlay({
	element: element,
	positioning: 'bottom-center',
	stopEvent: false
});
map.addOverlay(popup);

map.on('click', function(evt) {
	var feature = map.forEachFeatureAtPixel(evt.pixel,
		function(feature, layer) {
			return feature;
		});

	if (feature) {
		var geometry = feature.getGeometry();
		var coord = geometry.getCoordinates();
		popup.setPosition(coord);
		$(element).popover({
			'placement': 'top',
			'html': true,
			'content': "<h4>商品:" + feature.get('name') + "</h4>" + '' +
			'<div class="button icon-left ion-ios-telephone button-calm button-outline">' +
			'<a ng-href="tel: {{result.phone_number}}">' + feature.get('phone') + '</a> </div>' +
				"<p class='icon-left ion-ios-navigate'> " + feature.get('distance') + "公里</p>"
		});
		$(element).popover('show');
	} else {
		$(element).popover('destroy');
	}
});
```

当用户点击时，调用Bootstrap的Popover来显示信息。

![ElasticSearch Map](./images/elasticsearch_ionit_map.jpg)
    