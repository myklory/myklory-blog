title: 使用Django开发博客（2）--创建项目
date: 2015-09-23 9:02:21
tags: [Django]
---
上次章我们搭建了Django的开发环境，这一章就着手开始搭建一个博客。
<!--more-->
### **生成项目**
要使用Django框架，首先需要生成一个项目，在这个项目上搭建我们的博客。
生成项目命令
```bash
django-admin.py startproject amje
```
我这里的项目名称为amje。Django自动在当前目录生成amje这个目录，可以使用下面这个命令来查看一下amje目录的结构
```bash
tree amje
```
进入amje目录，可以发现在根目录下有manage.py这个文件，以后我们执行的Django命令都通过这个文件来执行。关于此文件和Django命令的详细介绍参考[**这里**](https://docs.djangoproject.com/en/1.8/ref/django-admin/)。
### **运行项目**
现在我们可以通过Django命令runserver来运行我们的项目。
```bash
python manage.py runserver
```
使用浏览器打开这个地址[**http://127.0.0.1:8000**](http://127.0.0.1:8000),可以看到我们的Django项目已经运行起来了。8000是Django默认的端口。
如果我们需要其他电脑可以访问项目，需要在runserver命令加上参数
```bash
python manage.py runserver 0.0.0.0:8000
```
现在就可以在其他电脑上使用IP:8000访问项目了。
### **生成APP**
在上一步中，我们生成了一个Django项目，并且没定一行代码就可以在浏览器中访问我们的项目。现在，我们要生成博客APP。一个Django可以包含多个APP。
```bash
python manage.py startapp algblog
```
我的博客APP取名为algblog，现在可以使用tree命令再次查看amje目录树。
### **使用views生成主页**
在第二节我们访问的主页是由Django自已生成的，在这一节我们生成自己的主页。
使用文本编辑器打开algblog/views.py，增加以下内容
```python
from django.http import HttpResponse

#主页view，在我们的主页上简单显示Hello algblog字符串
def index(request):
    return HttpResponse('Hello algblog')
```
现在我们有了主页的view，访问[**http://127.0.0.1:8000**](http://127.0.0.1:8000)发现主页仍然是Django默认页面，浏览器并没有显示我们期望的页面，浏览器怎么才能找到这个页面并且显示呢？答案是在amje/urls.py文件中，这个文件是Django是路由文件，在这个文件中增加路由。
```python
urlpatterns = [
    url(r'^admin/',include(admin.site.urls)),
    url(r'^$', 'algblog.views.index'),#这一行是新增的algblog主页路由
```
现在我们再访问[**http://127.0.0.1:8000**](http://127.0.0.1:8000)，发现主页上已经正确显示了Hello algblog。
