编辑-发布-分离的博客系统
===

概况
---

### 背景: 编辑-发布-开发分离

在这种情形中，编辑能否完成工作就不依赖于网站——脱稿又少了 个借口。这时候网站出错的概率太小了——你不需要一个缓存服务器、HTTP服务器，由于没有动态生成的内容，你也不需要守护进程。这些内容都是静态文件，你可以将他们放在任何可以提供静态文件托管的地方——CloudFront、S3等等。或者你再相信自己的服务器，Nginx可是全球第二好（第一还没出现）的静态文件服务器。

开发人员只在需要的时候去修改网站的一些内容。

So，你可能会担心如果这时候修改的东西有问题了怎么办。

1. 使用这种模式就意味着你需要有测试来覆盖这些构建工具、生成工具。
2. 相比于自己的代码，别人的CMS更可靠？

需要注意的是如果你上一次构建成功，你生成的文件都是正常的，那么你只需要回滚开发相关的代码即可。旧的代码仍然可以工作得很好。

其次，由于生成的是静态文件，查错的成本就比较低。

最后，重新放上之前的静态文件。
  
> 动态网页是下一个要解决的难题。我们从数据库中读取数据，再用动态去渲染出一个静态页面，并且缓存服务器来缓存这个页面。既然我们都可以用Varnish、Squid这样的软件来缓存页面——表明它们可以是静态的，为什么不考虑直接使用静态网页呢？

为了实现之前说到的``编辑-发布-开发分离``的CMS，我还是花了两天的时间打造了一个面向普通用户的编辑器。效果截图如下所示：

![Echeveria Editor](./images/eche-editor-screenshot.png)

作为一个普通用户，这是一个很简单的软件。除了Electron + Node.js + React作了一个140M左右的软件，尽管打包完只有40M左右 ，但是还是会把用户吓跑的。不过作为一个快速构建的原型已经很不错了——构建速度很快、并且运行良好。

尽管这个界面看上去还是稍微复杂了一下，还在试着想办法将链接名和日期去掉——问题是为什么会有这两个东西？

#### 从Schema到数据库

我们在我们数据库中定义好了Schema——对一个数据库的结构描述。在《[编辑-发布-开发分离](https://www.phodal.com/blog/editing-publishing-coding-seperate/)
》一文中我们说到了echeveria-content的一个数据文件如下所示：

```javascript
  {
    "title": "白米粥",
    "author": "白米粥",
    "url": "baimizhou",
    "date": "2015-10-21",
    "description": "# Blog post \n  > This is an example blog post \n Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. ",
    "blogpost": "# Blog post \n  > This is an example blog post \n Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. \n Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum."
  }
```

比起之前的直接生成静态页面这里的数据就是更有意思地一步了，我们从数据库读取数据就是为了生成一个JSON文件。何不直接以JSON的形式存储文件呢？

我们都定义了这每篇文章的基本元素:

1. title
2. author
3. date
4. description
5. content
6. url

即使我们使用NoSQL我们也很难逃离这种模式。我们定义这些数据，为了在使用的时候更方便。存储这些数据只是这个过程中的一部分，下部分就是取出这些数据并对他们进行过滤，取出我们需要的数据。

Web的骨架就是这么简单，当然APP也是如此。难的地方在于存储怎样的数据，返回怎样的数据。不同的网站存储着不同的数据，如淘宝存储的是商品的信息，Google存储着各种网站的数据——人们需要不同的方式去存储这些数据，为了更好地存储衍生了更多的数据存储方案——于是有了GFS、Haystack等等。运营型网站想尽办法为最后一公里努力着，成长型的网站一直在想着怎样更好的返回数据，从更好的用户体验到机器学习。而数据则是这个过程中不变的东西。

尽管，我已经想了很多办法去尽可能减少元素——在最开始的版本里只有标题和内容。然而为了满足我们在数据库中定义的结构，不得不造出来这么多对于一般用户不友好的字段。如链接名是为了存储的文件名而存在的，即这个链接名在最后会变成文件名：

```javascript
repo.write('master', 'contents/' + data.url + '.json', stringifyData, 'Robot: add article ' + data.title, options, function (err, data) {
      if(data.commit){
        that.setState({message: "上传成功" + JSON.stringify(data)});
        that.refs.snackbar.show();
        that.setState({
          sending: 0
        });
      }
    });
```    

然后，上面的数据就会变成一个对象存储到“数据库”中。

今天 ，仍然有很多人用Word、Excel来存储数据。因为对于他们来说，这些软件更为直接，他们简单地操作一下就可以对数据进行排序、筛选。数据以怎样的形式存储并不重要，重要的是他们都以文件的形式存储着。

#### git作为NoSQL数据库

在控制台中运行一下 ``man git``你会得到下面的结果:

![Man Git](./images/man-git.png)

这个答案看起来很有意思——不过这看上去似乎无关主题。

不同的数据库会以不同的形式存储到文件中去。blob是git中最为基本的存储单位，我们的每个content都是一个blob。redis可以以rdb文件的形式存储到文件系统中。完成一个CMS，我们并不需要那么多的查询功能。

> 这些上千年的组织机构，只想让人们知道他们想要说的东西。

我们使用NoSQL是因为：

1. 不使用关系模型
2. 在集群中运行良好
3. 开源
4. 无模式
5. 数据交换格式

我想其中只有两点对于我来说是比较重要的``集群``与``数据格式``。但是集群和数据格式都不是我们要考虑的问题。。。

我们也不存在数据格式的问题、开源的问题，什么问题都没有。。除了，我们之前说到的查询——但是这是可以解决的问题，我们甚至可以返回不同的历史版本的。在这一点上git做得很好，他不会像WordPress那样存储多个版本。

#### git + JSON文件

JSON文件 + Nginx就可以变成这样一个合理的API，甚至是运行方式。我们可以对其进行增、删、改、查，尽管就当前来说查需要一个额外的软件来执行，但是为了实现一个用得比较少的功能，而去花费大把的时间可能就是在浪费。

git的“API”提供了丰富的增、删、改功能——你需要commit就可以了。我们所要做的就是:

1. git commit
2. git push
Carrot使用了下面的方案来搭建他们的静态内容的CMS。

![Carrot](./images/carrot.png)

在这个方案里内容是用Contentful来发布他们的内容。而在我司[ThoughtWorks](https://www.thoughtworks.com/)的官网里则采用了Github来管理这些内容。于是如果让我们写一个基于Github的CMS，那么架构变成了这样：

![Github 编辑-发布-开发](./images/travis-edit-publish-code.png)

或许你也用过Hexo / Jekyll / Octopress这样的静态博客，他们的原理都是类似的。我们有一个代码库用于生成静态页面，然后这些静态页面会被PUSH到Github Pages上。

从我们设计系统的角度来说，我们会在Github上有三个代码库：

1. Content。用于存放编辑器生成的JSON文件，这样我们就可以GET这些资源，并用Backbone / Angular / React 这些前端框架来搭建SPA。
2. Code。开发者在这里存放他们的代码，如主题、静态文件生成器、资源文件等等。
3. Builder。在这里它是运行于Travis CI上的一些脚本文件，用于Clone代码，并执行Code中的脚本。

以及一些额外的服务，当且仅当你有一些额外的功能需求的时候。

1. Extend Service。当我们需要搜索服务时，我们就需要这样的一些服务。如我正考虑使用Python的whoosh来完成这个功能，这时候我计划用Flask框架，但是只是计划中——因为没有合适的中间件。
2. Editor。相比于前面的那些知识这一步适合更重要，也就是为什么生成的格式是JSON而不是Markdown的原理。对于非程序员来说，要熟练掌握Markdown不是一件容易的事。于是，一个考虑中的方案就是使用 Electron + Node.js来生成API，最后通过GitHub API V3来实现上传。

So，这一个过程是如何进行的。

### 用户场景

整个过程的Pipeline如下所示：

1. 编辑使用他们的编辑器来编辑的内容并点击发布，然后这个内容就可以通过GitHub API上传到Content这个Repo里。
2. 这时候需要有一个WebHooks监测到了Content代码库的变化，便运行Builder这个代码库的Travis CI。
3. 这个Builder脚本首先，会设置一些基本的git配置。然后clone Content和Code的代码，接着运行构建命令，生成新的内容。
4. 然后Builder Commit内容，并PUSH内容。

这里还依赖于WebHook这个东西——还没想到一个合适的解决方案。下面，我们对里面的内容进行一些拆解，Content里面由于是JSON就不多解释了。

步骤
---

### Step 1: 构建工具

Github与Travis之间，可以做一个自动部署的工具。相信已经有很多人在Github上玩过这样的东西——先在Github上生成Token，然后用travis加密：

```bash
travis encrypt-file ssh_key --add
```

加密后的Key就会保存到``.travis.yml``文件里，然后就可以在Travis CI上push你的代码到Github上了。

接着，你需要创建个deploy脚本，并且在``after_success``执行它：

```yml
after_success:
  - test $TRAVIS_PULL_REQUEST == "false" && test $TRAVIS_BRANCH == "master" && bash deploy.sh
```

在这个脚本里，你所需要做的就是clone content和code中的代码，并执行code中的生成脚本，生成新的内容后，提交代码。

```
#!/bin/bash

set -o errexit -o nounset

rev=$(git rev-parse --short HEAD)

cd stage/

git init
git config user.name "Robot"
git config user.email "robot@phodal.com"

git remote add upstream "https://$GH_TOKEN@github.com/phodal-archive/echeveria-deploy.git"
git fetch upstream
git reset upstream/gh-pages

git clone https://github.com/phodal-archive/echeveria-deploy code
git clone https://github.com/phodal-archive/echeveria-content content
pwd
cp -a content/contents code/content

cd code

npm install
npm install grunt-cli -g
grunt 
mv dest/* ../
cd ../
rm -rf code
rm -rf content

touch .

if [ ! -f CNAME ]; then
    echo "deploy.baimizhou.net" > CNAME
fi

git add -A .
git commit -m "rebuild pages at ${rev}"
git push -q upstream HEAD:gh-pages
```

这就是这个builder做的事情——其中最主要的一个任务是``grunt``，它所做的就是:

```javascript
grunt.registerTask('default', ['clean', 'assemble', 'copy']);
```

### Step 2: 静态页面生成

Assemble是一个使用Node.js，Grunt.js，Gulp，Yeoman 等来实现的静态网页生成系统。这样的生成器有很多，Zurb Foundation, Zurb Ink, Less.js / lesscss.org, Topcoat, Web Experience Toolkit等组织都使用这个工具来生成。这个工具似乎上个Release在一年多以前，现在正在开始0.6。虽然，这并不重要，但是还是顺便一说。

我们所要做的就是在我们的``Gruntfile.js``中写相应的生成代码。

```javascript
	assemble: {
      options: {
        flatten: true,
        partials: ['templates/includes/*.hbs'],
        layoutdir: 'templates/layouts',
        data: 'content/blogs.json',
        layout: 'default.hbs'
      },
      site: {
        files: {'dest/': ['templates/*.hbs']}
      },
      blogs: {
        options: {
          flatten: true,
          layoutdir: 'templates/layouts',
          data: 'content/*.json',
          partials: ['templates/includes/*.hbs'],
          pages: pages
        },
        files: [
          { dest: './dest/blog/', src: '!*' }
        ]
      }
    }
```    

配置中的site用于生成页面相关的内容，blogs则可以根据json文件的文件名生成对就的html文件存储到blog目录中。

生成后的目录结果如下图所示：

```
 .
├── about.html
├── blog
│   ├── blog-posts.html
│   └── blogs.html
├── blog.html
├── css
│   ├── images
│   │   └── banner.jpg
│   └── style.css
├── index.html
└── js
    ├── jquery.min.js
    └── script.js

7 directories, 30 files
```

这里的静态文件内容就是最后我们要发布的内容。

还需要做的一件事情就是：

```javascript
grunt.registerTask('dev', ['default', 'connect:server', 'watch:site']);
```

用于开发阶段这样的代码就够了，这个和你使用WebPack + React 似乎相差不了多少。
