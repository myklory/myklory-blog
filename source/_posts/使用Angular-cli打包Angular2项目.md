title: 使用Angular-cli打包Angular2项目
date: 2016-10-08 09:25:54
tags: [angular2]
---

我们在使用Angular2项目时，直接使用官网提供的基础配置文件，在NodeJS下面就可以生成一个新的ng2项目，但是这样非常不便利，每次都要新建目录，复制配置文件，使用Node命令安装支持库，最后才是写代码。Angular-cli就是用来简化这一操作的，而且官方配置文件不包含打包命令，对于新手来说，对System打包和webpack打包都不熟悉的情况下，使用Angular-cli能够非常方便的生存一样ng2项目，打包ng2项目，集中在编写项目代码上，省略繁琐的配置过程。
<!--more-->
#### **安装Angular-cli**
安装Angular-cli非常简单，就像安装其它Node库一样，我们一般安装在全局中：
```shell
npm install -g angular-cli
```
-g命令表示安装在全局。
下面就是使用常用的Angular-cli命令了
#### **新建ng2项目**
如果在要把现有目录转成ng2项目，只需要运行下面命令：
```shell
mkdir ng2-test
cd ng2-test
ng init
```
也可以使用new命令来新建一个项目，并在其中运行init命令。即把上面三个步骤合成一个：
```shell
ng new ng2-test
```
这样就建好了一个新的ng2项目。
#### **运行ng2项目**
在测试环境中运行ng2项目使用serve命令：
```shell
ng serve
```
也可以使用一般的npm命令：
```shell
npm start
```
然后在浏览器中打开[http://localhost:4200](http://localhost:4200)，即可查看效果。
可以使用-p,-h命令来指定主机和端口。
```shell
ng serve -h 0.0.0.0 -p 3000
```
#### **新建组件**
Angular-cli提供了命令方便的新建一个组件，省略了原来的手工新建组件工作。比如要新建一个组件。
```shell
ng g Component NewTest
```
上面使用会在new-test目录中新建一个组件，并且会生成模板文件html，组件文件ts，样式文件css。非常方便。
#### **新建service**
新建服务和新建组件一样：
```shell
ng g Service NewSer
```
#### **打包**
使用下面命令生成可发布的ng2文件：
```shell
ng build
```
在项目目录下就会生成dist目录，将里面的文件发布到服务器上就完成了项目的打包和布署。
上面只是简单的介绍了Angular-cli命令，可以使用下面命令查看所有的命令：
```shell
ng --help
```
或者使用下面命令查看对应的子命令：
```shell
ng --help generate
```
上面命令可以查看所有g命令可以生成的模板。