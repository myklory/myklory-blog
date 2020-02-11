---
title: Hexo和github搭建静态blog
date: 2020-02-11 13:06:09
tags: [hexo, github, blog]
categories: hexo
---

使用hexo和github搭建blog的好处就不在多说，这里主要记录一下如何搭建和如何配置
### 注册Github和生成github pages仓库
注册：略
#### 生成github pages仓库
新建仓库，名字为 xxx.github.io(xxx是你的github用户名)，在Setting中的Custom domain可以设置自己的域名。
这时，基于github pages的博客仓库就建好了。
### 安装nodejs
[nodejs](http://www.nodejs.org)
### 安装hexo
```bash
npm install -g hexo-cli
```
如果安装慢可以安装淘宝的cnpm
```bash
npm install -g cnpm --registry=https://registry.npm.taobao.org
```
### 创建hexo博客
新建一个博客存储目录，进入目录后执行以下命令:
```bash
hexo init #如果不是新建，而是从git克隆，则不需要这一句
npm install
```

使用下面命令在/public目录下生成静态网页文件:
```bash
hexo g
```
出错的话会给出哪个文件出错，不会生成那篇文章的静态文件。成功会在/public目录下生成github需要的文件 。
完成后就可以在本地测试环境是否搭建成功，执行下面命令启动一个本地服务器:
```bash
hexo s
```
访问给出的链接即可查看生成的博客

### 发布
配置根目录下的__config.yml,修改或增加以下内容:
```yml
deploy:
  type: git
  repo: https://github.com/myklory/myklory.github.io.git #替换成你的github博客地址
```
这样就将github和你的hexo关联进来，要同步/public目录中的文件到github只需要执行下面命令：
```bash
hexo deploy
```
如果不成功可能需要安装hexo-deployer-git：
```bash
npm install hexo-deployer-git
```

### 使用自定义域名
在hexo的source目录下新建CNAME文件,文件名需要大写,在其中添加自定义域名(blog.lnnl.top),执行hexo生成和发布命令,现在github博客仓库设置中可以看见github page的域名已经发生变化。
下一步配置阿里云的dns解析，具体配置见下图：
7牛云图片挂了，不想截图
需要注意的是在原github博客地址(myklory.github.io)后面要加上一个”.”。
配置完成后稍等几分钟访问blog.lnnl.top就可以打开本站。

### 主题
这里使用的是butterfly主题，使用方式参考[https://github.com/Molunerfinn/hexo-theme-melody](https://github.com/Molunerfinn/hexo-theme-melody)