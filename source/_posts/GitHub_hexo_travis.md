---
title: hexo github travis_ci 构建博客
date: 2017-11-22
author: 杨金坤
tags:
  - 其他
categories:
  - github
thumbnail: img/thumbnail/1.jpg
blogexcerpt:
    hexo github travis_ci 构建博客
---
    
# hexo github travis_ci 构建博客
## Travis CI
Travis CI 是目前新兴的开源持续集成构建项目，它与jenkins，GO的很明显的特别在于采用yaml格式，同时他是在在线的服务，不像jenkins需要你本地打架服务器，简洁清新独树一帜。目前大多数的github项目都已经移入到Travis CI的构建队列中，据说Travis CI每天运行超过4000次完整构建。对于做开源项目或者github的使用者，如果你的项目还没有加入Travis CI构建队列，那么我真的想对你说out了。

## 框架
Hexo搭建-->Github源码托管-->Travis自动构建-->Github提供的Gitpage服务托管
## 环境
* 安装[Nodejs](https://nodejs.org)
* 安装[Git](https://git-for-windows.github.io/)
* 申请[GitHub](https://github.com)
* 申请[Travis](https://travis-ci.org)
## Hexo
Node和Git都安装好后,首先创建一个文件夹,如blog,用户存放hexo的配置文件,然后进入blog里安装Hexo。

执行如下命令安装Hexo：


```
sudo npm install -g hexo
```


初始化然后，执行init命令初始化hexo,命令：


```
hexo init
```


生成静态页面


```
hexo generate
```


本地启动

启动本地服务，进行文章预览调试，命令：


```
hexo server
```


浏览器输入http://localhost:4000
## github
生成后的HTML文件和博客的源文件我是放到一个仓库的，只是使用了不同的分支

master：博客的静态文件，也就是hexo生成后的HTML文件，因为要使用Gitpage服务，所以他规定的网页文件必须是在master分支
blog-source：是博客的源代码

git提交

```
git checkout blog-source
git commit -m "blog"
git remote show
git add source/_posts/GitHub_hexo_travis.md
git remote add origin https://github.com/blog/blog.github.io.git
git push origin blog-source:blog-source
```

## Travis
* My Repositories旁边的+，意思是添加一个要自动构建的仓库
* 在Travis CI配置Github的Access Token
* 创建.travis.yml配置文件


参考文档
1. [HEXO+Github,搭建属于自己的博客](http://www.jianshu.com/p/465830080ea9)
2. [手把手教你使用Travis CI自动部署你的Hexo博客到Github上](http://blog.csdn.net/woblog/article/details/51319364)