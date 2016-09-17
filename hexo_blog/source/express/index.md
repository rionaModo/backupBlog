---
title: express 原理介绍之http服务
date: 2016-09-03 14:53:23
tags:
---
最近公司在用node做页面的渲染，于是开始学习起来node，凡是后台语言，我想肯定是少不了框架的，在node中运用比较普遍的我想可能就是express了吧。
Express 是一种保持最低程度规模的灵活 Node.js Web 应用程序框架，为 Web 和移动应用程序提供一组强大的功能。

### 采用http模块实现HTTP 服务器功能###
express的http请求是基于http模块实现，http模块实现的是基于事件的 HTTP 服务器


    var http = require('http');//http模块
    //请求的统一入口函数
    function serverHandle(request, response) {
        //http请求统一处理函数
        `` //middleware();``//根据request.url做处理
        console.log('host:'+request.headers.host+",url:"+request.url);
    }
    var server=http.createServer(serverHandle);//http模块的HTTP服务器对象
    server.listen(8000,function(){
        //监听成功后回调
        console.log('Server running at http://127.0.0.1:8000/');
        response.end('Hello World\n');
    })


以上是express的实现的基础，express中的中间件便是在 *serverHandle*进行处理的，启动项目时用app.use将所需要的中间件按代码执行顺序加入中间件序列，当监听到请求就把中间件按照加入的顺序在serverHandle执行。

## 才用express实现 HTTP 服务器功能 ##

    var express = require('express');
    var app = express(); // 对象
    //通过app.use() ,app.METHOD()加入 middleware 处理函数
    app.use('/', function (request, response) {
        response.send('Hello World!');
    });
    app.get('/', function (request, response) {
        response.send('Hello World!');
    });
    var server = app.listen(8000, function () {    //监听 8000端口
        var host = server.address().address;
        var port = server.address().port;

        console.log('Example app listening at http://%s:%s', host, port);
    });

middleware 即监听请求后服务器的执行的一系列处理方法，express采用app.use() ,app.METHOD()加入一些在服务器监听请求后的处理函数，为一个函数都可用response.send('Hello World!');等相关方法直接响应客服端，即实现 原生的response.end('Hello World!');方法, 为了让中间件与响应客服端做一定区分express4.x以及路由模块化等，express将响应客服端抽象出来var express = require('express'); var router = express.Router();  采用router实现


    var express = require('express');
    var router = express.Router();
    // 该路由使用的中间件
    router.use(function timeLog(req, res, next) {
      console.log('Time: ', Date.now());
      next();
    });
    // 定义网站主页的路由
    router.get('/', function(req, res) {
      res.send('Birds home page');
    });
    // 定义 about 页面的路由
    router.get('/about', function(req, res) {
      res.send('About birds');
    });
