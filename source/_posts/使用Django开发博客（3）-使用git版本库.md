title: 使用Django开发博客（3）--使用git版本库
date: 2015-09-23 10:05:53
tags: [Django]
---
在上一章我们实现了一个简单的页面，这一章中为我们的项目添加git管理。
详细的git文档请看[**这里**](https://git-scm.com/docs)。
<!--more-->
### **使用本地git管理项目**
#### **初始化版本库**
初始化git，在amje目录下使用命令
```bash
git init
```
这样就初始化了一个空的git版本库,使用下面命令来看来版本库状态。
```bash
git status
```
#### **忽略特定文件**
在继续使用git之前，我们需要确认哪些文件是不需要使用git进行版本管理的，比如生成的中间文件和其他自动生成的文件等。需要把这些文件排除在外，在amje根目录添加.gitignore文件，在这个文件中添加需要忽略的文件或者目录，幸运的是，我们不需要手动添加常用的需要忽略的文件，github已经为我们准备好了，可以在[**这里**](https://github.com/github/gitignore)找到对应语言的.gitignore文件。我们这里使用python的.gitignore文件，将它的内容复制到amje的根目录的.gitignore文件中。


#### **增加README.md**
在amje是根目录下增加README.md文件，这个文件的内容需要使用markdown语法来编写，使用github时作为说明文档显示。
#### **添加文件到版本库**
使用以下命令把未忽略的文件纳入版本库管理
```bash
git add .
```
.表示所有文件，也可以指定具体的文件名。
#### **提交修改到本地**
使用以下命令提交修改到本地
```bash
git commit -m "初始化Django项目"
```
引号内是本次修改的描述。

再次使用git status命令查看版本库状态，可以发现命令行提示没有文件需要提交。
### **使用远程git库**
上一节只是使用了本地的git版本库进行管理，要使用github等远程库应该怎么做呢？
#### **建立远程库**
在github上建立新的版本库，名字输入amje，不要选择Initialize this repository with a README选项，建立完成后可以看到以下内容。
```bash
…or create a new repository on the command line
echo # amje >> README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/xyzaabb11/amje.git
git push -u origin master

…or push an existing repository from the command line
git remote add origin https://github.com/xyzaabb11/amje.git
git push -u origin master

…or import code from another repository
You can initialize this repository with code from a Subversion, Mercurial, or TFS project
```
这段内容给出了新建版本库，导入现有版本库和从其他版本库导入的方法，新建版本库基本和本章内容一致。
#### **建立本地和远程版本库的连接**
使用以下命令：
```bash
git remote add origin https://github.com/xyzaabb11/amje.git
```
以后可以使用origin代表远程库。
#### **提交本地修改到远程库**
使用命令：
```bash
git push -u origin master
```
根据提示输入用户名和密码，每次使用commit命令提交本地库之后，即可使用这个命令推送到远程库，其中master是分支名，要推送其他分支使用对应的分支名即可。
再次打开github中是项目主页，可以看见我们的项目文件和README.md文件的显示
