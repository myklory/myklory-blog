title: centos环境配置2--python3.5
date: 2016-07-28 16:28:08
tags: [CentOS, Python35]
---
上一文章配置了基本的LNMP环境，这次就进行python环境的配置。本着用新不用旧的原则，我选用了python3.
<!--more-->
#### **安装python3**

准备编译环境

```shell
sudo yum groupinstall 'Development Tools'
sudo yum install zlib-devel bzip2-devel  openssl-devel ncurses-devel
```

下载python3.5.2源代码

```shell
wget https://www.python.org/ftp/python/3.5.2/Python-3.5.2.tgz
```

解压

```shell
tar zxf Python-3.5.2.tgz
```

编译安装到/usr/python3.5目录下,使用下面命令进行配置，编译，安装。

```shell
sudo mkdir /usr/python3.5
./configure --prefix=/usr/python3.5
make
sudo make install
```

然后需要把python3.5连接到/usr/bin目录下

```shell
sudo ln -s /usr/python3.5/bin/python3 /usr/bin/python3
sudo ln -s /usr/python3.5/bin/pip3 /usr/bin/pip3
```

至此，python3安装完毕。

#### **安装virtualenv**

```shell
sudo pip install virtualenv
```

然后我们就可以建立一个虚拟的python环境，在这个环境下安装的python库不会影响真实的环境。

```shell
virtualenv --python=python3 flask
```

参数--python指明虚拟环境要使用的python版本， flask指在flask这个目录下建立虚拟环境。

然后使用source 命令进入虚拟环境

```shell
source flask/bin/activate
```

使用deactivate命令退出虚拟环境。

在虚拟环境下和真实环境使用python一样。

python环境搭建完毕。

#### **测试**

安装flask库

```shell
pip install flask
```

在flask目录下新建app.py文件，内容如下：

```python
from flask import Flask
app = Flask(__name__)

app.route('/')
def hello_world():
    return 'Hello World'

if __name__ == '__main__':
    app.run(host='0.0.0.0')
```

然后在命令行中运行

```shell
python app.py
```

出现下面提示说明正确

```shell
* Running on http://0.0.0.0:5000/ (Press CTRL+C )
```

然后在浏览器中输入[http://115.28.152.100:5000](http://115.28.152.100:5000) 显示Hello World说明我们最简单的Web服务器已经搭建成功。

#### **安装ipython**

ipython用来代替默认的python命令行环境, 但是使用ipython会默认使用pyhton2.7, 我们需要使用pip3来安装.

```shell
sudo pip3 install ipython
sudo ln -s /usr/python3.5/bin/ipython3 /usr/bin/ipython3
```

然后使用ipython3就可以进行python3.5的环境.
