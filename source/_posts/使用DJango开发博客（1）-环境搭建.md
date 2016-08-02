title: 使用DJango开发博客（1）--环境搭建
date: 2015-09-22 14:07:54
tags: [Django]
---
拥有一个自己的网站一直是我的一个想法，为了实现这个想法学过PHP，才入门就放弃了，无他，学不懂。这个想法就一直搁浅，直到最近开始接触python,发现python也可以用来开发网站，而且有非常成熟的框架Django，也有相对轻量级的flask,由于我才接触网站开发，需要快速的实现各种功能，Django就满足了我的需要，可以进行快速开发。等到我需要了解网站的实现原理的时候再来学习flask也不迟。
<!--more-->
### **开发环境**
```python
{
    'OS':'ubuntu 15.04',
    'Pyhton':'3.4', #我接触python较晚，就直接学习3.4了,没考虑过2.7，至于版本之争，我始终坚持满足需要的就是最好的。
    'Django':'1.8.4',
}
```
### **环境搭建**
#### **安装虚拟环境**
[**virtualenv**](https://virtualenv.pypa.io)可以用来建立我们的Django开发环境，而不与其他的环境产生冲突，这里我建议对基于不同框架或库的项目建立不同的虚拟环境，尽量不对其他项目产生影响。使用下面命令来安装virtualenv
```Bash
sudo apt-get install virtualenv
```
或
```Bash
pip install virtualenv
```
#### **创建虚拟环境**
详细的virtualenv命令请看[**这里**](https://virtualenv.pypa.io/en/latest/reference.html),我们使用这条命令来创建我们的虚拟环境
```Bash
virtualenv --python=python3 ~/env/django
```
#### **激活和退出虚拟环境**
激活虚拟环境
```Bash
source ~/env/django/bin/activate
```
终端的提示符出现如下字样表示虚拟环境激活成功
```Bash
(django)namine@namine:~$
```
可以看到在普通的提示符前增加了(django)字样
需要回到普通的终端使用以下命令来退出虚拟环境
```bash
deactivate
```
#### **安装Django**
安装Django比较简单，可使用pip或easy_install命令来安装
```bash
pip install django
```
由于我们是在虚拟环境中安装的Django，对普通环境下的python并没有产生影响，在普通环境下并不能使用Django命令。
### **结束语**
至此，我们的环境配置就算完结，在linux下是非常简单的，只需要几个命令，如同我写下这段文字一样简单。在Windows下这是一个非常繁琐的事情，一大堆文件等着下载。这也是我在两个系统上都进行了配置后选择linux进行开发的原因。
