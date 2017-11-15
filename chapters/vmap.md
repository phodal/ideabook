GeoJSON与ElasticSearch实现高级图形搜索
===

概况
---

### Showcase

在线Demo见： [http://vmap.phodal.com/](http://vmap.phodal.com/)

或者你已经使用过了相应多的省市区与地图联动，但是这些联动往往是单向的、不可逆。并且这些数据往往都是在线使用的，不能离线使用。下图是一个结合百度地图的省市区与地图联动：

![一般的省市区与地图联动](./images/general-province-city-map.png)

我们可以在这个应用里选择，相应的省市区然后地图会跳转到相应的地图。当我们在地图上漫游的时候，如果没有显示当前的省市区是不是变得很难使用。于是，我们就来创建一个吧：

![地图到省市区联动](./images/anti-map-action.jpg)

### jQuery + Mustache + Leaflet

相关技术栈：

 - Bootstrap，UI显示~~，地球人都知道。
 - jQuery，Bootstrap依赖。
 - Require.js，模块化。
 - Mustache，模板生成。
 - Leaflet，交互地图库。

步骤
---

### Step 1: 离线地图与搜索

在GitHub上搜索数据的过程中，发现了一个名为[d3js-geojson](https://github.com/ufoe/d3js-geojson)的项目里面放着中国详细省、市、县数据，并且还有及GeoJSON文件。

这就意味着两件事：

 - 地图离线
 - 多边形搜索

首先，我们要知道GeoJSON是怎样的一个存在。

> GeoJSON是一种对各种地理数据结构进行编码的格式，基于Javascript对象表示法的地理空间信息数据交换格式。GeoJSON对象可以表示几何、特征或者特征集合。GeoJSON支持下面几何类型：点、线、面、多点、多线、多面和几何集合。GeoJSON里的特征包含一个几何对象和其他属性，特征集合表示一系列特征。

换句话来说，根据这个文件里面的多边形，我们可以绘制出中国地图。由于上面的点是真实的地理位置信息，所以无论我们怎样的缩放这些点的位置都不会发生变化。如下图是GitHub对这个数据文件的解析：

![中国GeoJSON文件](./images/china-geojson.jpg)

（PS: 预览可以打开这个页面：[Vmap GeoJSON](https://github.com/phodal/vmap/blob/gh-pages/static/data/china.json)

当然这似乎不是一个专业人员维护的数据，所以存在一些偏差。但是这些数据意味着，我们不需要依靠于在线地图就可以完成大部分的功能了。在线地图一直都是一个缓慢的存在，并且Google Map在多数人那都是不可用的。

接着问题来了，我们并没有把每个用户的数据存入到数据库中，那么我们怎么才能实现搜索？

#### 多边形搜索

所谓的多边形搜索就是画一个圈圈（任意多边形），然后你就可以去约这个圈圈里的人，如下图所示：

![多边形搜索](./images/geopoly2d-small.png)

而圈圈搜索依赖于圈圈上的连续的点构建的形状来进行搜索，上面的每个点都包含了相应的经纬度。因此，只要是在这个圈圈里的用户都是可以搜索得到的。

这样实现的前提是：

 - 要有一个支持多边形搜索的搜索引擎，如ElasticSearch、Solr、MongoDB等等。
 - 要将用户的数据成功地存成GEO信息。

详细信息可以见: [VMap Bot](https://github.com/phodal/vmap-bot)


### Step 2: 从地点到地图上显示

拿Bootstrap实现一个Dropdown是一件很容易的事，我们只要动用一下相应的模板就好了。难就难在，如果去与地图交互。

最初的时候要用Event的形式来实现，但是发现这样似乎会让其紧耦合。就改用了监听Hash Change的形式来实现，在总的地图上每一个省都有一个对应的ID，这个ID会对应相应的省的数据。如下图所示：

![Province Hash](./images/province-hash-with-map.jpg)

接着，我们就需要从这个Hash中判断它的级别和ID，随后转由相应的函数来处理这些逻辑即可。随后，我们要做两件事：

 - 创建对应省的市的Dropdown
 - 从地图上跳转到省

创建对应省的市的Dropdown，我们只需要根据地点重新生成一个新的Menu再插入即可。

从地图上跳转到对应的省的时候：

 1. 用Ajax请求获取这个省的GeoJSON文件
 2. 获取这个市的中心位置，并对其进行缩放
 3. 将上面的每个市绘制到地图上

在这个过程中遇到的最大的坑是：中国有北京、上海、天津、重庆等直辖市，还有港、澳等自治区（PS：台湾是一个省）。对于这些特殊的地点，那么的缩放级别肯定会更高。

同理，我们也可以对上面的市运行处理。但是因为这些市并不存在GEO信息，所以我只是从其多连形信息取了一个点，再将这个点放到data-geo中：

![Data GEO](./images/city-with-geo.jpg)

对应于省市的，对于区的处理也是如此。这样，我们就完成了地点到地图的显示了。

###Step 3: 从地图到地点上显示

从地图上到地点就比较简单了，点击时修改对应的text即可。

![VMap Click ](./images/vmap-click-handler.jpg)

