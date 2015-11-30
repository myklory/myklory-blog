title: 使用Hexo在github上搭建博客
date: 2015-11-30 15:42:47
tags: github hexo
---
### **1. 注册github和生成博客空间**
略
### **2. 安装Nodejs和hexo**
#### **2.1 安装Nodejs**
[Node.JS](www.nodejs.org)
#### **2.2 安装Hexo**
NodeJS安装好后使用下面命令安装Hexo:
```bash
npm install hexo-cli -g
npm install hexo --save
```
新建一个博客存储目录，进入目录后执行以下命令:
```bash
hexo init '如果不是新建，而是从git克隆，则不需要这一句
npm install
```
使用下面命令在/public目录下生成静态网页文件:
```bash
hexo generate
```
出错的话会给出哪个文件出错，不会生成那篇文章的静态文件。成功会在/public目录下生成github需要的文件 。
完成后就可以在本地测试环境是否搭建成功，执行下面命令启动一个本地服务器:
```bash
hexo server
```
完成后在浏览器中打开[http://localhost:4000](http://localhost:4000)可以看到默认生成的helloworld页面。
### **3. 同步生成的静态页面到github**
配置根目录下的__config.yml,修改或增加以下内容:
```json
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
### **4. 主题**
暂略

