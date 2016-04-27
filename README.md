Phodal's Idea实战指南
===

虽然在我的[Idea](https://github.com/phodal/ideas)列表里已经有了足够多的Idea，但是这些Idea都没有一个好的实战指南。这个电子书的目标就是为这些Idea提供实战指南，一步步搭建。

我的其他电子书：

 - 《[一步步搭建物联网系统](https://github.com/phodal/designiot)》
 - 《[GitHub 漫游指南](https://github.com/phodal/github-roam)》
 - 《[RePractise](https://github.com/phodal/repractise)》
 - 《[Growth: 全栈增长工程师指南](https://github.com/phodal/growth-ebook)》

Idea指南的格式：

 * 概况
    * 背景
    * Showcase
    * 技术栈等介绍
 * 步骤
    * Step *
    * 练习建议          

欢迎关注我的微信公众号（扫描下面的二维码或搜索 Phodal）.

![QRCode](https://raw.githubusercontent.com/phodal/growth/master/www/img/wechat.jpg)

有钱捧个钱场:

![Alipay](https://raw.githubusercontent.com/phodal/growth/master/docs/alipay.png)![Wechat](https://raw.githubusercontent.com/phodal/growth/master/docs/wechat.png)

目录
---

*   [Hadoop + Pig 分析Nginx日志](#hadoop-pig-分析nginx日志)
*   [书籍录入程序](#书籍录入程序)
*   [程序员专属Badge制作](#程序员专属badge制作)
    *   [SVG与SVGWrite](#svg与svgwrite)
    *   [高级Badge](#高级badge)
    *   [最后代码](#最后代码)
*   [文本编辑器](#文本编辑器)
*   [JavaScript打造Slide应用](#javascript打造slide应用)
    *   [需求](#需求)
    *   [EchoesWorks功能](#echoesworks功能)
    *   [现状](#现状)
        *   [idea](#idea)
    *   [实现机制](#实现机制)
*   [编辑-发布-分离应用](#编辑-发布-分离应用)
    *   [系统架构](#系统架构)
        *   [用户场景](#用户场景)
    *   [Builder: 构建工具](#builder-构建工具)
    *   [Code: 静态页面生成](#code-静态页面生成)
    *   [编辑-发布-开发分离](#编辑-发布-开发分离)
    *   [从Schema到数据库](#从schema到数据库)
    *   [git作为NoSQL数据库](#git作为nosql数据库)
        *   [git + JSON文件](#git-json文件)
*   [GitHub数据分析](#github数据分析)
*   [Google Map与Solr实现多连形搜索](#google-map与solr实现多连形搜索)
    *   [Solr](#solr)
    *   [Gmap Solr Polygon 搜索实战](#gmap-solr-polygon-搜索实战)
        *   [Solr Flask](#solr-flask)
        *   [Google map Polygon](#google-map-polygon)
*   [Growth](#growth)
    *   [从Web到混合应用，再到桌面应用](#从web到混合应用再到桌面应用)
    *   [响应式设计](#响应式设计)
    *   [平台特定代码](#平台特定代码)
        *   [存储](#存储)
        *   [数据分析](#数据分析)
        *   [更新](#更新)
        *   [桌面应用](#桌面应用)
    *   [未来](#未来)
*   [Ionic ElasticSearch打造O2O应用](#ionic-elasticsearch打造o2o应用)
    *   [构架设计](#构架设计)
        *   [GIS架构说明 —— 服务端](#gis架构说明-服务端)
        *   [GIS架构说明 —— 客户端](#gis架构说明-客户端)
    *   [Django GIS准备](#django-gis准备)
    *   [配置Django](#配置django)
        *   [Django Haystack Model创建](#django-haystack-model创建)
        *   [创建search_index](#创建search_index)
    *   [Ionic ElasticSearch 创建页面](#ionic-elasticsearch-创建页面)
    *   [Ionic ElasticSearch Service](#ionic-elasticsearch-service)
        *   [添加OpenLayer 3](#添加openlayer-3)
    *   [Ionic OpenLayer 地图显示](#ionic-openlayer-地图显示)
*   [跨协议的物联网平台设计](#跨协议的物联网平台设计)
    *   [物联网层级结构](#物联网层级结构)
    *   [物联网服务层](#物联网服务层)
*   [一步步搭建JavaScript框架: Lettuce](#一步步搭建javascript框架-lettuce)
    *   [JavaScript项目名称](#javascript项目名称)
    *   [生成Javascript项目框架](#生成javascript项目框架)
        *   [安装Yeoman lib生成器](#安装yeoman-lib生成器)
    *   [Build JavaScript项目](#build-javascript项目)
*   [基于Virtual DOM的测试代码生成](#基于virtual-dom的测试代码生成)
    *   [Virtual DOM](#virtual-dom)
        *   [一个Jasmine jQuery测试](#一个jasmine-jquery测试)
        *   [virtual-dom与HyperScript](#virtual-dom与hyperscript)
    *   [标记DOM变化](#标记dom变化)
        *   [其他](#其他)
*   [移动框架](#移动框架)
    *   [构建框架](#构建框架)
    *   [RequireJS 使用](#requirejs-使用)
        *   [使用RequireJS](#使用requirejs)
        *   [环境准备](#环境准备)
        *   [添加路由](#添加路由)
        *   [创建主页View](#创建主页view)
        *   [jQuery Sidr](#jquery-sidr)
        *   [Django Tastypie示例](#django-tastypie示例)
        *   [跨域支持](#跨域支持)
        *   [整合](#整合)
        *   [RequireJS Plugins](#requirejs-plugins)
    *   [简单的博客构成](#简单的博客构成)
        *   [获取在线数据](#获取在线数据)
    *   [获取每篇博客](#获取每篇博客)
    *   [添加博文的路由](#添加博文的路由)
    *   [重构](#重构)
        *   [构建函数](#构建函数)
        *   [移动CMS滑动](#移动cms滑动)
*   [Oculus + Node.js + Three.js 打造VR世界](#oculus-node.js-three.js-打造vr世界)
    *   [Node Oculus Services](#node-oculus-services)
        *   [安装Node NMD](#安装node-nmd)
        *   [Node.js Oculus Hello，World](#node.js-oculus-helloworld)
        *   [Node Oculus WebSocket](#node-oculus-websocket)
    *   [Three.js + Oculus Effect + DK2 Control](#three.js-oculus-effect-dk2-control)
    *   [Three.js DK2Controls](#three.js-dk2controls)
        *   [欧拉角与四元数](#欧拉角与四元数)
        *   [Three.js DK2Controls](#three.js-dk2controls-1)
        *   [Three.js KeyHandler](#three.js-keyhandler)
*   [Python制作照片地图](#python制作照片地图)
    *   [CartoDB](#cartodb)
*   [React制作Markdown简历生成器](#react制作markdown简历生成器)
*   [D3.js 制作技能树](#d3.js-制作技能树)
    *   [从零开始设计技能树: 使用Graphviz建立模型](#从零开始设计技能树-使用graphviz建立模型)
    *   [Graphviz](#graphviz)
    *   [简单的技能树](#简单的技能树)
    *   [计算点数与Star Here](#计算点数与star-here)
        *   [从Knockout开始](#从knockout开始)
    *   [点数计算](#点数计算)
    *   [Tooltipster](#tooltipster)
        *   [D3.js Tooltipster Require配置](#d3.js-tooltipster-require配置)
        *   [整合代码](#整合代码)
    *   [结束](#结束)
*   [D3.js 技术栈](#d3.js-技术栈)
*   [文本转化为Logo](#文本转化为logo)
    *   [需求说明](#需求说明)
    *   [Python 文字转Logo实战](#python-文字转logo实战)
        *   [基础代码](#基础代码)
        *   [圆角代码](#圆角代码)
        *   [颜色配置](#颜色配置)
    *   [结束](#结束-1)
*   [游戏地图生成器](#游戏地图生成器)
*   [GEOJSON与ElasticSearch实现高级图形搜索](#geojson与elasticsearch实现高级图形搜索)
    *   [离线地图与搜索](#离线地图与搜索)
        *   [地图离线](#地图离线)
        *   [多边形搜索](#多边形搜索)
    *   [从地点到地图上显示](#从地点到地图上显示)
    *   [从地图到地点上显示](#从地图到地点上显示)
    *   [Demo](#demo)


License
---

[![Phodal's Article](http://brand.phodal.com/shields/article-small.svg)](https://www.phodal.com/) [![Phodal's Idea](http://brand.phodal.com/shields/idea-small.svg)](http://ideas.phodal.com/)

© 2015~2016 [Phodal Huang](https://www.phodal.com). This code is distributed under the Creative Commons Attribution-Noncommercial-No Derivative Works 3.0  License. See `LICENSE` in this directory.

[![待我代码编成,娶你为妻可好](http://brand.phodal.com/slogan/slogan.svg)](http://www.xuntayizhan.com/person/ji-ke-ai-qing-zhi-er-shi-dai-wo-dai-ma-bian-cheng-qu-ni-wei-qi-ke-hao-wan/)

