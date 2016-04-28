分析网站日志，打造访问地图
===

概况
---

### 背景

这个项目的背景是起源于，我有一个2G左右的网站访问日志。我想看看访问网站的人都来自哪里，于是我想开始想办法来分析这日志。当时正值大数据火热的时候，便想拿着Hadoop来做这样一件事。

### ShowCase

最后的效果如下图如示：

![Demo](./images/map_with_bg.jpg)

这是一个Web生成的界面，通过Elastic.js向搜索引擎查询数据，将再这些数据渲染到地图上。

###Hadoop + Pig + Jython + AmMap + ElasticSearch

我们使用的技术栈有上面这些，他们的简介如下：

 - Hadoop是一个由Apache基金会所开发的分布式系统基础架构。用户可以在不了解分布式底层细节的情况下，开发分布式程序。充分利用集群的威力进行高速运算和存储。
 - Pig 是一个基于Hadoop的大规模数据分析平台，它提供的SQL-LIKE语言叫Pig Latin，该语言的编译器会把类SQL的数据分析请求转换为一系列经过优化处理的MapReduce运算。
 - Jython是一种完整的语言，而不是一个Java翻译器或仅仅是一个Python编译器，它是一个Python语言在Java中的完全实现。Jython也有很多从CPython中继承的模块库。
 - AmMap是用于创建交互式Flash地图的工具。您可以使用此工具来显示您的办公室地点，您的行程路线，创建您的经销商地图等。
 - ElasticSearch是一个基于Lucene 构建的开源，分布式，RESTful 搜索引擎。 设计用于云计算中，能够达到搜索实时、稳定、可靠和快速，并且安装使用方便。

步骤
---

总的步骤并不是很复杂，可以分为：

 - 搭建基础设施
 - 解析access.log
 - 转换IP为GEO信息
 - 展示数据到地图上

###Step 1: 环境搭建

在这一些系列的实战中，比较麻烦的就是安装这些工具，我们需要安装上面提到的一系列工具。对于不同的系统来说，都有相似的安装工具：

 - Windows上可以使用Chocolatey
 - Ubuntu / Mint上可以使用aptitude
 - CentOS / OpenSUSE上可以使用yum安装
 - Mac OS上可以使用brew安装

如下是Mac OS下安装Hadoop、Pig、Elasticsearch、Jython的方式

```bash
brew install hadoop
brew install pig 
brew install elasticsearch
brew install jython
```

对于其他操作系统也可以使用相似的方法来安装。接着我们还需要安装一个Hadoop的插件，用于连接Hadoop和ElasticSearch。

下载地址：[https://github.com/elastic/elasticsearch-hadoop](https://github.com/elastic/elasticsearch-hadoop)

复制其中的``elasticsearch-hadoop-*.jar``、``elasticsearch-hadoop-pig-*.jar``到你的pig库的目录，如我的是：``/usr/local/Cellar/pig/0.14.0``。

由于我使用提早期的版本，所以这里我的文件名是：``elasticsearch-hadoop-2.1.0.Beta3.jar``、``elasticsearch-hadoop-pig-2.1.0.Beta3.jar``。

下面我们就可以尝试去解析我们的日志了。

###Step 2: 解析 access.log

在开始解析之前，先让我们来看看几条Nginx的日志：

```
106.39.113.203 - - [28/Apr/2016:10:40:31 +0000] "GET / HTTP/2.0" 200 0 "https://www.phodal.com/" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/49.0.2623.87 Safari/537.36" -
66.249.65.119 - - [28/Apr/2016:10:40:51 +0000] "GET /set_device/default/?next=/blog/use-falcon-peewee-build-high-performance-restful-services-wordpress/ HTTP/1.1" 302 5 "-" "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" -
```

而上面的日志实际上是有对应的格式的，这个格式写在我们的Nginx配置文件中。如下是上面的日志的格式：

```
log_format  access $remote_addr - $remote_user [$time_local] "$request" '
		'$status $body_bytes_sent "$http_referer" '
		'"$http_user_agent" $http_x_forwarded_for';
```		

在最前面的是访问者的IP地址，然后是访问者的当地时间、请求的类型、状态码、访问的URL、用户的User Agent等等。随后，我们就可以针对上面的格式编写相应的程序，这些代码如下所示：

```
register file:/usr/local/Cellar/pig/0.14.0/libexec/lib/piggybank.jar;
register file:/usr/local/Cellar/pig/0.14.0/libexec/lib/elasticsearch-hadoop-pig-2.1.0.Beta3.jar;

RAW_LOGS = LOAD 'data/access.log' USING TextLoader as (line:chararray);

LOGS_BASE = FOREACH RAW_LOGS GENERATE
    FLATTEN(
      REGEX_EXTRACT_ALL(line, '(\\S+) - - \\[([^\\[]+)\\]\\s+"([^"]+)"\\s+(\\d+)\\s+(\\d+)\\s+"([^"]+)"\\s+"([^"]+)"\\s+-')
    )
    AS (
        ip: chararray,
        timestamp: chararray,
        url: chararray,
        status: chararray,
        bytes: chararray,
        referrer: chararray,
        useragent: chararray
    );

A = FOREACH LOGS_BASE GENERATE ToDate(timestamp, 'dd/MMM/yyyy:HH:mm:ss Z') as date, ip, url,(int)status,(int)bytes,referrer,useragent;
--B = GROUP A BY (timestamp);
--C = FOREACH B GENERATE FLATTEN(group) as (timestamp), COUNT(A) as count;
--D = ORDER C BY timestamp,count desc;
STORE A INTO 'nginx/log' USING org.elasticsearch.hadoop.pig.EsStorage();
```

在第1~2行里，我们使用了自定义的jar文件。接着在第4行，载入了log文件，并其值赋予RAW_LOGS。随后的第6行里，我们取出RAW_LOGS中的每一个值 ，根据下面的正则表达式，取出其对应的值到对象里，如``- -``前面的(\\S+)对应的是ip，最后将这些值赋给LOGS_BASE。

接着，我们就可以对值进行一些特殊的处理，如A是转化时间戳后的结果。B是按时间戳排序后的结果。最后，我们再将这些值存储到ElasticSearch对应的索引``nginx/log``中。

###Step 3: 转换IP

```
register file:/usr/local/Cellar/pig/0.14.0/libexec/lib/piggybank.jar;
register file:/usr/local/Cellar/pig/0.14.0/libexec/lib/elasticsearch-hadoop-pig-2.1.0.Beta3.jar;
register utils.py using jython as utils;

RAW_LOGS = LOAD 'data/access.log' USING TextLoader as (line:chararray);

LOGS_BASE = FOREACH RAW_LOGS GENERATE
    FLATTEN(
      REGEX_EXTRACT_ALL(line, '(\\S+) - - \\[([^\\[]+)\\]\\s+"([^"]+)"\\s+(\\d+)\\s+(\\d+)\\s+"([^"]+)"\\s+"([^"]+)"\\s+-')
    )
    AS (
        ip: chararray,
        timestamp: chararray,
        url: chararray,
        status: chararray,
        bytes: chararray,
        referrer: chararray,
        useragent: chararray
    );

A = FOREACH LOGS_BASE GENERATE ToDate(timestamp, 'dd/MMM/yyyy:HH:mm:ss Z') as date, utils.get_country(ip) as country,
    utils.get_city(ip) as city, utils.get_geo(ip) as location,ip,
    utils.query(url) as url,(int)status,(int)bytes,referrer,useragent;

STORE A INTO 'nginx/log' USING org.elasticsearch.hadoop.pig.EsStorage();
```

```
#!/usr/bin/python

import sys
sys.path.append('/Users/fdhuang/test/lib/python2.7/site-packages/')
import pygeoip
gi = pygeoip.GeoIP("data/GeoLiteCity.dat")

@outputSchema('everything:chararray')
def query(url):
    try:
        return url
    except:
        pass

@outputSchema('city:chararray')
def get_city(ip):
    try:
        city = gi.record_by_name(ip)["city"]
        return city
    except:
        pass


@outputSchema('country:chararray')
def get_country(ip):
    try:
        city = gi.record_by_name(ip)["country_name"]
        return city
    except:
        pass

@outputSchema('location:chararray')
def get_geo(ip):
    try:
        geo = str(gi.record_by_name(ip)["longitude"]) + "," + str(gi.record_by_name(ip)["latitude"])
        return geo
    except:
        pass
```        

###Step 4: 展示

```javascript
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

var view = new ol.View({
	center: [12119653.781323666,4054689.6824535457],
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
	renderer: exampleNS.getRendererFromQueryString(),
	target: 'map',
	view: view
});

var client = new $.es.Client({
	hosts: 'localhost:9200'
});

var query = {
	index: 'nginx',
	type: 'log',
	size: 200,
	body: {
		query: {
			query_string: {
				query: "*"
			}
		},
		aggs: {
			2: {
				terms: {
					field: "city",
					size: 10,
					order: {
						_count: "desc"
					}
				}
			}
		}
	}
};

client.search(query).then(function(results){
	var vectorSource = new ol.source.Vector({ });
	$.each(results.hits.hits, function(index, result){
		result = result._source;
		if(result.location === null) {
			return;
		}
		var position = result.location.split(",");
		var pos = ol.proj.transform([parseFloat(position[0]), parseFloat(position[1])], 'EPSG:4326', 'EPSG:3857');

		var iconFeature = new ol.Feature({
			geometry: new ol.geom.Point(pos),
			city: result.city
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
				'content': "<h4>" + feature.get('city') + "</h4>"
			});
			$(element).popover('show');
		} else {
			$(element).popover('destroy');
		}
	});

	var width = 300,
		height = 300,
		radius = Math.min(width, height) / 2;

	var color = d3.scale.ordinal()
		.range(["#1abc9c", "#16a085", "#2ecc71", "#27ae60", "#4caf50", "#8bc34a", "#cddc39", "#3498db", "#2980b9", "#34495e", "#2c3e50", "#2196f3", "#03a9f4", "#00bcd4", "#009688", "#e74c3c", "#c0392b", "#f44336", "#e67e22", "#d35400", "#f39c12", "#ff9800", "#ff5722", "#ffc107", "#f1c40f", "#ffeb3b", "#9b59b6", "#8e44ad", "#9c27b0", "#673ab7", "#e91e63", "#3f51b5", "#795548", "#9e9e9e", "#607d8b", "#7f8c8d", "#95a5a6", "#bdc3c7", "#ecf0f1", "#efefef"]);

	var arc = d3.svg.arc()
		.outerRadius(radius - 10)
		.innerRadius(0);

	var pie = d3.layout.pie()
		.sort(null)
		.value(function(d) { return d.doc_count; });

	var svg = d3.select("body").append("svg")
		.attr("width", width)
		.attr("height", height)
		.append("g")
		.attr("transform", "translate(" + width / 2 + "," + height / 2 + ")");

	var citys = results.aggregations[2].buckets;
	citys.forEach(function(city) {
		city.doc_count = +city.doc_count;
	});

	var g = svg.selectAll(".arc")
		.data(pie(citys))
		.enter().append("g")
		.attr("class", "arc");

	g.append("path")
		.attr("d", arc)
		.style("fill", function(city) { return color(city.data.key); });

	g.append("text")
		.attr("transform", function(d) { return "translate(" + arc.centroid(d) + ")"; })
		.attr("dy", ".35em")
		.style("text-anchor", "middle")
		.text(function(d) { return d.data.key; });

});
```

###练习建议
