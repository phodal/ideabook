一步步搭建JavaScript框架: Lettuce
===

概况
---

### 背景

从开始打算写一个MV*，到一个简单的demo，花了几天的时间，虽然很多代码都是复制/改造过来的，然而**It Works**(nginx的那句话会让人激动有木有)。现在他叫lettuce，代码 [https://github.com/phodal/lettuce](https://github.com/phodal/lettuce)，如果有兴趣可以加入我们。

虽然js还不够expert，但是开始了。

步骤
---

###Step 1: 注册npm和bower包

一开始我做的3次commits是:

    * e4e6e04 - Add README.md (3 weeks ago) <Fengda HUANG>
    * 37411d7 - publish bower (3 weeks ago) <Fengda HUANG>
    * aabf278 - init project (3 weeks ago) <Fengda HUANG>

是的一开始什么也没做，除了从``bower``和``npm``上注册了一个叫做``lettuce``的库:

```javascript
{
  "name": "lettuce",
  "version": "0.0.2",
  "authors": [
    "Fengda HUANG <h@phodal.com>"
  ],
  "description": "A Mobile JavaScript Framework",
  "main": "index.js",
  "moduleType": [
    "amd",
    "node"
  ],
  "keywords": [
    "lettuce",
    "mobile"
  ],
  "license": "MIT",
  "homepage": "http://lettuce.phodal.com",
  "private": false,
  "ignore": [
    "**/.*",
    "node_modules",
    "bower_components",
    "test",
    "tests"
  ]
}
```

然后在我们还没有开始写代码的时候版本就已经是``0.0.2``这个速度好快。。总结如下:

 - 取一个好的名字
 - 在npm和bower上挖一个坑给自己
 - 开始写README.md

所以我的``README.md``是这样子的

```
#Lettuce

> A is Mobile JavaScript Framework

Coming soon
```

是的，我们的代码已经``Coming soon``了。

### Step 2: 生成Javascript项目框架

为了简化这一个痛苦的过程，我们还是用yeoman。

#### 安装Yeoman lib生成器

1.安装yeoman

```bash
npm install -g yo
```

2.安装generator-lib

```bash
npm install -g generator-lib
```

3.创建项目

```bash 
mkdir ~/lettuce && cd $_
yo lib
```

接着我们就迎来了

```
     _-----_
    |       |
    |--(o)--|   .--------------------------.
   `---------´  |    Welcome to Yeoman,    |
    ( _´U`_ )   |   ladies and gentlemen!  |
    /___A___\   '__________________________'
     |  ~  |
   __'.___.'__
 ´   `  |° ´ Y `

[?] What do you want to call your lib? Lettuce
[?] Describe your lib: A Framework for Romantic
[?] What is your GitHub username? phodal
[?] What is your full name? Fengda Huang
[?] What year for the copyright? 2015
```

省略上百字，你的目录里就会有

```
.
|____.editorconfig
|____.gitattributes
|____.gitignore
|____.jshintrc
|____bower.json
|____demo
| |____assets
| | |____main.css
| | |____normalize.css
| |____index.html
|____dist
| |____Lettuce.js
| |____Lettuce.min.js
|____docs
| |____MAIN.md
|____Gruntfile.js
|____index.html
|____LICENSE.txt
|____package.json
|____README.md
|____src
| |_____intro.js
| |_____outro.js
| |____main.js
|____test
| |____all.html
| |____all.js
| |____lib
| | |____qunit.css
| | |____qunit.js
```

这么多的文件。

#### Build JavaScript项目

于是我们执行了一下

    grunt

就有了这么多的log:

```
Running "concat:dist" (concat) task
File "dist/Lettuce.js" created.

Running "jshint:files" (jshint) task
>> 1 file lint free.

Running "qunit:files" (qunit) task
Testing test/all.html .OK
>> 1 assertions passed (20ms)

Running "uglify:dist" (uglify) task
File "dist/Lettuce.min.js" created.

Done, without errors.
```

看看我们的Lettuce.js里面有什么

```
(function(root, undefined) {
  "use strict";
/* Lettuce main */
// Base function.
var Lettuce = function() {
  // Add functionality here.
  return true;
};
// Version.
Lettuce.VERSION = '0.0.1';
// Export to the root, which is probably `window`.
root.Lettuce = Lettuce;
}(this));
```

我们的库写在[立即执行函数表达式](https://www.phodal.com/blog/javascript-immediately-invoked-function-expression)里面。这样便是和jQuery等库一样了。

grunt里的任务包含了:

 - jshint 代码检查
 - contact 合并js到一个文件
 - minify js 压缩js
 - qunit 单元测试

这样我们就可以轻松上路了。

### Step 3: 寻找所需要的函数

### Step 4: 整合

### Step 5: 测试

### 练习建议
