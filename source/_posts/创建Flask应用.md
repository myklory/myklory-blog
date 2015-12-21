title: 创建Flask应用
date: 2015-12-17 10:09:15
tags: [flask, python]
---
## **环境**
首先需要安装Python环境，Python现在有2.7和3.4版本。这里使用的3.4版本。安装python过程相对比较简单，不再详细叙述，直接开始我们的flask应用环境搭建。
<!--more-->
安装**virtualenv**:
```shell
pip install virtualenv
```
如果使用Ubuntu需要使用以下命令安装:
```shell
sudo apt-get install python-virtualenv
```
新建一个目录iEng，进入到这个目录使用下面的命令创建一个虚拟环境：
```shell
virtualenv --python=python3 flask
```
进入虚拟环境：
```shell
source flask/bin/activate
```
安装flask：
```shell
pip install flask
```
## **文件结构**
在iEng目录下建立下面的文件目录：
```shell
mkdir app
mkdir app/static
mkdir app/templates
```
我们应用程序放在app目录下，static目录下放css、js文件和静态图片文件，templates目录下放模板文件。
## **Hello, Flask!**
创建app/__init__.py文件让我们的app目录成为一个模块，在__init__.py文件里添加flask初始代码：
```python
from flask import Flask

app = Flask(__name__)
from app import views
```
上面代码首先导入flask模块，接着创建flask的应用对象app，最后导入views模块。
视图是响应来自网页浏览器请求的处理器。在Flask中，视图是python函数，每一个视图函数映射一个或多个请求的URL。
让我们在app/view.py中编写第一个视图函数：
```python
from app import app
@app.route('/')
def index():
    return 'Hello, Flask!'
```
route装饰器创建了从网址'/'到这个函数的映射。index返回一个字符串。
最后一步是创建一个启动脚本，用来启动我们的flask应用。在iEng根目录创建run.py：
```python
from app import app
app.run(debug = True, host = '0.0.0.0')
```
debug参数表示可以在网页中显示调试信息，host设为'0.0.0.0'表示可以在其他计算机上访问。
## **运行应用**
在iEng根目录下运行命令：
```shell
python run.py
```
如无错误访问[http://127.0.0.1:5000](http://127.0.0.1:5000)网址即可显示"Hello, Flask!"。
