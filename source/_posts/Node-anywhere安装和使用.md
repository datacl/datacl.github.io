---
title: Node-anywhere 安装和使用
date: 2018-11-01
author: 杨金坤
tags:
  - js
categories:
  - nodejs
thumbnail: img/thumbnail/1.jpg
blogexcerpt:
    Node-anywhere搭建了一个本地服务器环境，随时随地预览本地静态资源文件。
---

# Node-anywhere

Node-anywhere搭建了一个本地服务器环境，随时随地预览本地静态资源文件。

 

如果电脑没有安装nodejs的同学,首先去nodeJs官网下载最新版nodeJs     https://nodejs.org/en/

 

安装成功后win+r打开cmd 输入node -help 或者node -v查看是否安装成功 

 

装好后输入 npm install anywhere -g来安装anywhere。注意如果是mac系统会提示你权限不够，需要在代码前加上 sudo获取管理员权限。即sudo npm install anywhere -g。

 

想要以某个路径作为静态文件服务器的根目录分享，只需要在该目录下执行：

anywhere 
 

默认不添加 -s 命令会在命令敲击后，同时打开浏览器访问 http://ipv4地址:8000/ 这个路径。

anywhere -p 8000 // 指定静态服务器的端口号  
anywhere -s // 静默执行，不打开浏览器  
我们需要搭建一个简易的本地服务器，Nodejs就可以满足我们这个需求
