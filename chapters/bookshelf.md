书籍录入移动应用：条形码扫描
===

概况
---

### 背景

这个项目的起源是我想录入我的书架上的书籍——当时，大概有近四百本左右。由于大部分的手机软件都是收费的，或封闭的，因此我便想着自己写一个app来完成书籍的录入。

### ShowCase

最后的效果如下图所示：

![Bookshelf](./images/bookshelf.jpg)

代码见： [https://github.com/phodal/bookshelf/](https://github.com/phodal/bookshelf/)

### Ionic + Zxing

所需要的移动框架还是Ionic，用于扫描条形码的库是ZXing。

步骤
---

开始之前，我们需要先安装Ionic，并且使用它来创建一个APP。然后我们还需要添加对应的二维码扫描库，代码如下所示：

```bash
phonegap plugin add phonegap-plugin-barcodescanner
```
7
接着我们就可以开始制作我们的APP了。

###Step 1: ZXing扫描与Douban API

我们需要在我们的模板里，添加一个ICON或者按钮来触发程序调用相应的函数：

```html
<i class="icon icon-right ion-qr-scanner" ng-click="scan()"></i>
```

在我们的函数里，我们只需要调用cordovaBarcodeScanner的scan方法就可以获取到二维码的值。再用$http.get去获取豆瓣API的相应的结果，并且将这个结果存储到数据库中。代码如下所示：

```javascript
$scope.scan = function () {
  $cordovaBarcodeScanner
    .scan()
    .then(function (barcodeData) {
      $scope.info = barcodeData.text;
      $http.get("https://api.douban.com/v2/book/isbn/" + barcodeData.text).success(function (data) {
        $scope.detail = data;
        saveToDatabase(data, barcodeData);
      });
    }, function (error) {
      alert(error);
    });
}
```    

随后，我们就可以创建我们的代码来保存数据到数据库中。

###Step 2: 存储数据库

开始之前，我们需要添加Cordova的SQLite插件：

```bash
cordova plugin add https://github.com/litehelpers/Cordova-sqlite-storage.git
```

在系统初始化的时候，创建对应的数据库及其表。

```javascript
if(window.cordova) {
  //$cordovaSQLite.deleteDB("my.db");
  db = $cordovaSQLite.openDB("my.db");
} else {
  db = window.openDatabase("my.db", "1.0", "bookshelf", -1);
}
$cordovaSQLite.execute(db, "CREATE TABLE IF NOT EXISTS bookshelf (id integer primary key, title text, image text, publisher text, author text, isbn text, summary text)");
```

接着，我们就可以上一步获取的数据取出相应的字段，调用bookshelfDB服务将其存储到数据库中。

```javascript
function saveToDatabase(data, barcodeData) {
  bookshelfDB.add({
    title: data.title,
    image: data.image,
    publisher: data.publisher,
    author: data.author,
    summary: data.summary,
    isbn: barcodeData.text
  });
}
```    

下面就是我们的bookshelfDB服务，我们实现了get、add、remove、update，即CRUD。

```javascript
.factory('bookshelfDB', function($cordovaSQLite, DBA) {
	var self = this;
	self.all = function() {
		return DBA.query("SELECT id, title, image, publisher, author, isbn, summary FROM bookshelf")
			.then(function(result){
				return DBA.getAll(result);
			});
	};
	self.get = function(memberId) {
		var parameters = [memberId];
		return DBA.query("SELECT id, title, image, publisher, author, isbn, summary FROM bookshelf WHERE id = (?)", parameters)
			.then(function(result) {
				return DBA.getById(result);
			});
	};
	self.add = function(member) {
		var parameters = [member.id, member.title, member.image, member.publisher, member.author, member.isbn, member.summary];
		return DBA.query("INSERT INTO bookshelf (id, title, image, publisher, author, isbn, summary) VALUES (?,?,?,?,?,?,?)", parameters);
	};
	self.remove = function(member) {
		var parameters = [member.id];
		return DBA.query("DELETE FROM bookshelf WHERE id = (?)", parameters);
	};
	self.update = function(origMember, editMember) {
		var parameters = [editMember.id, editMember.title, origMember.id];
		return DBA.query("UPDATE bookshelf SET id = (?), title = (?) WHERE id = (?)", parameters);
	};
	return self;
})
```	

### 上传数据

### 练习建议
