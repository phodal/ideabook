Solr实现多边形地理搜索
===

概况
---

### 背景

### Showcase

![Google Map Solr](./images/gmap-solr.jpg)

GitHub [https://github.com/phodal/gmap-solr](https://github.com/phodal/gmap-solr)

### Solr

> Solr是一个高性能，采用Java5开发，基于Lucene的全文搜索服务器。同时对其进行了扩展，提供了比Lucene更为丰富的查询语言，同时实现了可配置、可扩展并对查询性能进行了优化，并且提供了一个完善的功能管理界面，是一款非常优秀的全文搜索引擎。

简单地说: 它是一个搜索引擎

> 文档通过Http利用XML 加到一个搜索集合中。查询该集合也是通过http收到一个XML/JSON响应来实现。它的主要特性包括：高效、灵活的缓存功能，垂直搜索功能，高亮显示搜索结果，通过索引复制来提高可用性，提供一套强大Data Schema来定义字段，类型和设置文本分析，提供基于Web的管理界面等。

即schema.xml

**Solr 安装**

```bash
brew install solr
```

步骤
---

用Flask搭建一个简单的servrices，接着在前台用google的api，对后台发出请求。

### Step 1: Solr Flask

由于，直接调用的是Solr的接口，所以我们的代码显得比较简单:

```python
class All(Resource):
    @staticmethod
    def get():
        base_url = ''
        url = (base_url + 'select?q=' + request.query_string + '+&wt=json&indent=true')
        result = requests.get(url)
        return (result.json()['response']['docs']), 201, {'Access-Control-Allow-Origin': '*'}


api.add_resource(All, '/geo/')
```

我们在前台需要做的便是，组装geo query。

###Step 2: Google map Polygon

在Google Map的API是支持Polygon搜索的，有对应的一个

```javascript
google.maps.event.addListener(drawingManager, 'polygoncomplete', renderMarker);
```

函数来监听，完成``polygoncomplete``时执行的函数，当我们完成搜索时，便执行``renderMarker``，在里面做的便是:

```javascript
$.get('/geo/?' + query, function (results) {
    for (var i = 0; i < results.length; i++) {
        var location = results[i].geo[0].split(',');
        var myLatLng = new google.maps.LatLng(location[0], location[1]);
        var title = results[i].title;
        marker = new google.maps.Marker({
            position: myLatLng,
            map: map,
            title: title
        });

        contentString = '<h1>City</h1><br/> address ' + i + '';

        google.maps.event.addListener(marker, 'click', (function (marker, contentString, infowindow) {
            return function () {
                infowindow.setContent(contentString);
                infowindow.open(map, marker);
            };
        })(marker, contentString, infowindow));
    }
});
```

对应的去解析数据，并显示在地图上
   