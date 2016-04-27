书籍录入程序
======

概况
---

### 背景

### ShowCase

![Bookshelf](./images/bookshelf.jpg)

代码： [https://github.com/phodal/bookshelf/](https://github.com/phodal/bookshelf/)

### Ionic + Zxing

步骤
---

    phonegap plugin add phonegap-plugin-barcodescanner


###Step 1: ZXing扫描与Douban API

```
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

###Step 2: 存储数据库

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

```javascript
if(window.cordova) {
  //$cordovaSQLite.deleteDB("my.db");
  db = $cordovaSQLite.openDB("my.db");
} else {
  db = window.openDatabase("my.db", "1.0", "bookshelf", -1);
}
$cordovaSQLite.execute(db, "CREATE TABLE IF NOT EXISTS bookshelf (id integer primary key, title text, image text, publisher text, author text, isbn text, summary text)");
```

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

###练习建议
