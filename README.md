# hbs-test
hbs-test-source: https://github.com/donpark/hbs.git

### 模板引擎 handlebars
前端常用模板引擎handlebars，因此我服务器也选择handlebars当模板引擎，便于统一管理

### 服务器安装handlebars
服务器我选择nodejs来开发，因此handlebars选用hbs  
以下是基于nodejs操作，请先安装nodejs,npm等等

	npm install hbs --save-dev
安装hbs到自己项目中，并写入包配置文件

### hbs的layout问题
服务器采用experss框架来搭建，内置渲染layout,文件名layout.something,something代表任何后缀名  
本测试为了方便阅读开发，将采用html为后缀名  
设置本项目开始路径为/,有如下文件
/views/layout.html 

	<html>
		<head>
		<meta http-equiv="Content-Type" content="text/html; charset=utf-8" /> 
			<title>无标题</title>
		</head>
		<body>
			{{{body}}}
			<hr/>
            注意标题-title
			我在layout页面
		</body>
	</html>
其中"{{{body}}}"会替换成你需要渲染的页面文件，暂时"{{{body}}}"为固定，不予改动

### 配置启动文件,并测试express的layout文件
/index.js

	var express = require("express")
	, path = require("path")
	, hbs = require("hbs");

	var app = express();

	app.set("view engine", "html"); // 方便阅读设置为html
	app.set("views", path.join(__dirname+"/views"));
	app.engine("html", hbs.__express);
    
	app.get("/", function(req, res){
		res.render("index", {
			title: "首页",
            index_value: "这里是首页提供的变量值lalala"
		})
	})
	app.get("/test/", function(req, res){
		res.render("test", {
			title: "测试页面",
            test_value: "这是测试给出的一个变量值lalala"
		})
	})

	app.listen(3000);
	console.log("listen to 3000");
/views/index.html

	<h1>我在首页</h1>
	<hr/>
    <p>服务器给我的值：{{index_value}}</p>
	<p>这里是首页啊</p>
/views/test.html
	
    <h1>我在测试页</h1>
    <hr/>
    <p>服务器给我的值：{{test_value}}</p>
    <p>这里是测试页啊</p>
    
+ 访问地址：首页：127.0.0.1:3000/  
![首页](http://localhost:3000/public/img/index.jpg)  
+ 访问地址：测试页：127.0.0.1:3000/test/
![测试](http://localhost:3000/public/img/test.jpg)  

### 自定义hbs的模块文件
本人喜欢的模块有: (模块存放路径/views/block/*)
+ head_start.html head头部模板，主要用来控制meta
+ head_end.html head尾部模板，主要用来加载css、js
+ body_banner.html banner模板
+ body_header.html 头部模板
+ body_footer.html 尾部模板

views/block/head_start.html

	<head>
		<meta charset="utf-8">
views/block/head_end.html

		<link rel="stylesheet" type="text/css" href="/public/css/style.css">
		<script src="/public/js/jquery.min.js"></script>
	</head>
views/block/body_start.html
	
    <body>
        <header class="header">
            <div>
                <nav class="nav">
                    <a href="/">首页</a>
                    <a href="/test/">测试页面</a>
                </nav>
            </div>
        </header>
views/block/body_banner.html

		<div class="banner"></div>

views/block/body_footer.html

		<footer class="footer">Copyright (C) 2015-2015 *.*, All rights reserved</footer>
	</body>
定义完文件，开始配置hbs的模块控制
在/index.js的添加：  
(注意：此处的模块读取是缓存到服务器，修改模块后需要重启服务器，如需自动读取最新模块，请自己写自动读取模块的程序~~~~)

    hbs.registerPartial("head_start", fs.readFileSync(__dirname+"/views/block/head_start.html", "utf-8"));
    hbs.registerPartial("head_end", fs.readFileSync(__dirname+"/views/block/head_end.html", "utf-8"));
    hbs.registerPartial("body_header", fs.readFileSync(__dirname+"/views/block/body_header.html", "utf-8"));
    hbs.registerPartial("body_banner", fs.readFileSync(__dirname+"/views/block/body_banner.html", "utf-8"));
    hbs.registerPartial("body_footer", fs.readFileSync(__dirname+"/views/block/body_footer.html", "utf-8"));
hbs模块配置完成后，开始调用，将在layout调用，重构layout

	<!doctype html>
	<html>
		{{> head_start}}
			<title>{{title}}</title>
		{{> head_end}}
		{{> body_header}}
		{{> body_banner}}
			<div class="wrap">
				{{{body}}}
				<hr/>
				注意标题-title
				我在layout页面
			</div>
		{{> body_footer}}
	</html>
/public/css/style.css文件自己随便写的，就不贴代码出来了
+ 访问地址：首页：127.0.0.1:3000/  
![首页](http://localhost:3000/public/img/block_index.jpg)  
+ 访问地址：测试页：127.0.0.1:3000/test/
![测试](http://localhost:3000/public/img/block_test.jpg)  

### 自定义hbs的模块目录
首先定义模块目录下的文件：随便定义4个如下：
/views/blockdirs/block_1.html
	
    <p>我是block_1的值</p>
/views/blockdirs/block_2.html
	
    <p>我是block_2的值</p>
/views/blockdirs/block_3.html
	
    <p>我是block_3的值</p>
/views/blockdirs/block_4.html
	
    <p>我是block_4的值</p>
在/index.js的添加：  

	hbs.registerPartials(__dirname+"/views/blockdirs");
(注意：此处的模块目录是缓存到服务器，修改模块目录下文件后需要重启服务器，如需自动读取最新模块，请自己写自动读取模块的程序~~~~)  
新建文件来加载模块目录里面文件，如下：  
/views/blockdirs.html

    <h1>我在模塊配置文件實例目錄，下面將引入blockdirs裡面的模塊</h1>
    <hr/>
    {{> block_1}}
    {{> block_2}}
    {{> block_3}}
    {{> block_4}}
为新文件增加新路由，在/index.js的添加：

    app.get("/blockdirs/", function(req, res){
        res.render("blockdirs", {
            title: "模块配置目录页面",
            test_value: "这是模块配置目录给出的一个变量值lalala"
        })
    })
+ 访问地址：首页：127.0.0.1:3000/blockdirs/  
![模块目录页面](http://localhost:3000/public/img/blockdirs.jpg)  

updatetime: 2015-01-27