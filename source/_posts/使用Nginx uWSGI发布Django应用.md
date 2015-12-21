title: 使用Nginx uWSGI发布Django应用
date: 2015-12-21 10:24:55
tags： [Nginx, uWSGI, Django]

---
### 目录
[TOC]

这一篇本来应该最后来写的，但是昨天在阿里云上看见学生可以用9.9的价格购买低配的阿里云服务器。1核CPU，1G内存，15G流量，40G/20G系统盘。对于我等来说已经足够使用了。立马用小妹的身份买了一个。一晚上时间配置好了Nginx+uWSGI的Django环境。
<!--more-->
### 1.为什么要用Nginx和uWSGI
使用uWSGI就可以实现Django的WEB服务，那为什么还要加上nginx呢？这篇[**文章**](http://lenciel.cn/2013/08/why-you-need-something-like-gunicorn/)说得比较清楚。
### **2.服务器环境**
阿里云的镜像市场里面只有Ubuntu的14.04版本，所以我们的操作都将在这个版本上进行。
### **3.安装Nginx,MariaDB,PhpMyAdmin**
由于我们的Django应用使用了MySQL，所以需要安装MariaDB和PhpMyAdmin，MariaDB可以看作是MySQL的替代产品，用法都一样。
#### **3.1 安装Nginx**
使用下面命令安装Nginx
```bash
sudo apt-get install nginx
```
安装完成后在浏览器中输入目标机器的IP，就可以看到nginx的欢迎页了。
增加php支持，打开文件/etc/nginx/sites-available/default，增加index.php，修改后如下
```python
server {
      listen 80 default_server;
      listen [::]:80 default_server ipv6only=on;
      root /usr/share/nginx/html;
      index index.php index.html index.htm;
      # Make site accessible from http://localhost/
      server_name server.unixmen.local
}
```
在此文件的下方找到location ~ \.php$ 一行，这一行是注释了的，现在去除注释并做如下修改
```python
location ~ \.php$ {
         try_files $uri =404;   #增加这一句
         fastcgi_split_path_info ^(.+\.php)(/.+)$;
         #       # NOTE: You should have "cgi.fix_pathinfo = 0;" in php.ini
         #
         #       # With php5-cgi alone:
         fastcgi_pass 127.0.0.1:9000;
         #       # With php5-fpm:
         #fastcgi_pass unix:/var/run/php5-fpm.sock;
         fastcgi_index index.php;
         include fastcgi.conf;
    }
```
使用下面命令测试配置是否通过
```shell
sudo nginx -t
```
#### **3.2 安装MariaDB**
使用下面命令安装MariaDB
```shell
sudo apt-get install mariadb-client mariadb-server
```
在安装过程中需要设置root帐户密码
#### **3.3 安装php**
```shell
sudo apt-get install php5 php5-fpm php5-mysql
```
打开php.ini
```shell
sudo vim /etc/php5/fpm/php.ini
```
找到cgi.fix_pathinfo=1这一行，去掉注释改成
```shell
cgi.fix_pathinfo=0
```
重启php服务
```shell
sudo service php5-fpm restart
```
下面测试php是否正常工作，编辑下面文件：
```shell
sudo vim /usr/share/nginx/html/info.php
```
输入以下内容
```php
<?php
    phpinfo();
?>
```
打开以下文件
```php
sudo vim /etc/php5/fpm/pool.d/www.conf
```
找到
```php
;listen = /var/run/php5-fpm.sock
```
修改为
```php
listen = 127.0.0.1:9000
```
重启php，nginx
```shell
sudo service php5-fpm restart
sudo service nginx restart
```
在浏览器中输入目标IP/info.php可以看到PHP的基本信息。
#### **3.4 安装PHPMyAdmin**
```shell
sudo apt-get install phpmyadmin
```
过程中会让你选择Apache或lighthttp，都可以无所谓，还会提示输入mysql的密码。
创建符号连接到nginx的目录：
```shell
sudo ln -s /usr/share/phpmyadmin/ /usr/share/nginx/html
```
即可在浏览器中通过IP/phpmyadmin访问管理mysql。
### **4. 配置Nginx uWSGI Django**
#### **4.1 配置Django开发环境**
详见Django系列文章
#### **4.2 安装uWSGI**
##### **4.2.1 安装**
在虚拟环境下安装uwsgi
```shell
sudo apt-get install python3.4-dev
pip install uwsgi
```
如果安装uwsgi出现错误：<font color='red'>setuptools must be installed to install from a source distribution</font>，请重新建立虚拟环境，再重新安装uwsgi。
##### **4.2.2 测试**
建立测试文件www/test.py：
```python
# test.py
def application(env, start_response):
    start_response('200 OK', [('Content-Type','text/html')])
    return [b"Hello World"] # python3
    #return ["Hello World"] # python2
```
运行uWSGI：
```shell
uwsgi --http :8000 --wsgi-file test.py
```
在浏览器访问8000端口，可以看到输出**Hello World**。
##### **4.2.3 测试Django项目**
我们使用amje项目来进行测试。目录为www/amje。
运行普通命令测试：
```shell
python manage.py runserver 0.0.0.0:8000
```
在浏览器访问显示正常。
使用uWSGI验证Django项目：
```shell
uwsgi --http :8000 --module amje.wsgi
```
#### **4.2.4 为Django项目配置Nginx**
在amje目录下新建文件uwsgi_params:
```python
uwsgi_param  QUERY_STRING       $query_string;
uwsgi_param  REQUEST_METHOD     $request_method;
uwsgi_param  CONTENT_TYPE       $content_type;
uwsgi_param  CONTENT_LENGTH     $content_length;

uwsgi_param  REQUEST_URI        $request_uri;
uwsgi_param  PATH_INFO          $document_uri;
uwsgi_param  DOCUMENT_ROOT      $document_root;
uwsgi_param  SERVER_PROTOCOL    $server_protocol;
uwsgi_param  REQUEST_SCHEME     $scheme;
uwsgi_param  HTTPS              $https if_not_empty;

uwsgi_param  REMOTE_ADDR        $remote_addr;
uwsgi_param  REMOTE_PORT        $remote_port;
uwsgi_param  SERVER_PORT        $server_port;
uwsgi_param  SERVER_NAME        $server_name;
```
在amje目录下新建amje_nginx.conf:
```python
# amje_nginx.conf

# the upstream component nginx needs to connect to
upstream django {
    # server unix:///home/namine/www/amje/amje.sock; # for a file socket
    server 127.0.0.1:8001; # for a web port socket (we'll use this first)
}

# configuration of the server
server {
    # the port your site will be served on
    listen      8000;
    # the domain name it will serve for
    server_name 127.0.0.1; # substitute your machine's IP address or FQDN
    charset     utf-8;

    # max upload size
    client_max_body_size 75M;   # adjust to taste

    # Django media
    location /media  {
        alias /home/namine/www/amje/media;  # your Django project's media files - amend as required
    }

    location /static {
        alias /home/namine/www/amje/static; # your Django project's static files - amend as required
    }

    # Finally, send all non-media requests to the Django server.
    location / {
        uwsgi_pass  django;
        include     /home/namine/www/amje/uwsgi_params; # the uwsgi_params file you installed
    }
}
```
这样就有了处理Django的nginx配置文件，下面创建到nginx配置目录的符号连接：
```shell
sudo ln -s ~/www/amje/amje_nginx.conf /etc/nginx/sites-enabled/
```
重启nginx：
```shell
sudo service nginx restart
```
##### **3.4.5 测试nginx和uwsgi运行test.py**
进入test.py所在的www目录，运行下面命令：
```shell
uwsgi --socket :8001 --wsgi-file test.py
```
如果没什么错误，浏览器正确显示**Hello World**。
这里的命令和**4.2.2**的命令有所不同。
这整个过程的大体意思就是浏览器通过8000端口发送请求到监听8000端口的nginx,nginx通过8001端口把请求转给uwsgi，uwsgi调用python对请求进行解释，最后再逆向返回浏览器。
##### **3.4.6 测试nginx和uwsgi运行Django项目**
进入amje目录，运行下面命令：
```shell
uwsgi --socket :8001 --module amje.wsgi
```
在浏览器中可以看到正确显示了amje的主页。
##### **3.4.7 配置uwsgi运行ini文件**
每次手动输入上面命令显示麻烦，也不好进行后续操作，现在写一个amje_uwsgi.ini文件用来启动uwsgi。
```ini
# amje_uwsgi.ini file
[uwsgi]

# Django-related settings
# the base directory (full path)
chdir           = /home/namine/www/amje
# Django's wsgi file
module          = amje.wsgi
# the virtualenv (full path)
home            = /home/namine/env/django

# process-related settings
# master
master          = true
# maximum number of worker processes
processes       = 10
# the socket (use the full path to be safe
socket          = 127.0.0.1:8001
# ... with appropriate permissions - may be needed
# chmod-socket    = 664
# clear environment on exit
vacuum          = true
```
运行下面命令：
```shell
uwsgi --ini amje_uwsgi.ini
```
在浏览器中可以看到正确显示了amje的主页。
##### **3.4.8 自动运行uwsgi**
如果每次都需要手动运行上面的命令来运行uwsgi显得很麻烦，有没有一开机就自动运行的方法呢？这里supervisor就出场了。
安装supervisor:
```shell
sudo apt-get install supervisor
```
Supervisor 的全局的配置文件位置在：
```shell
/etc/supervisor/supervisor.conf
```
正常情况下我们并不需要去对其作出任何的改动，只需要添加一个新的 *.conf 文件放在
```shell
/etc/supervisor/conf.d/
```
我们在这个目录下添加一个新的文件amje_supervisor.conf:
```shell
[program:amje]
# 启动命令入口
command=/home/namine/env/django/bin/uwsgi /home/namine/www/amje/amje_uwsgi.ini

# 命令程序所在目录
directory=/home/namine/www/amje
#运行命令的用户名
user=root
		
autostart=true
autorestart=true
#日志地址
stdout_logfile=/home/namine/www/amje/logs/uwsgi_supervisor.log
```
重启supersivor：
```shell
sudo service supersivor restart
```
这时会提示uwsgi_supervisor.log不存在，新建这个文件再重启。现在可以在浏览器里看到amje的主页了。
### **4.Django项目后台样式**
如果不配置Django的静态文件目录，会造成Admin后台样式丢失。需要进行以下操作修改。
在amje/setting.py文件下增加
```python
STATIC_ROOT = os.path.join(BASE_DIR, 'static')
```
与4.2.4节配置文件一致。
收集项目静态文件到此目录
```shell
python manage.py collectstatic
```
现在，可以正确的显示Django后台样式了。
### **4.结束语**
到这里为止，我们的配置就算完成，可以看见这是一个相当繁琐的过程，目录规划也不是很合理，但总算我们的Django已经跑起来了。计算机使人变懒，我们上面的步骤其实可以用脚本来实现，这个以后再说了。
现在可以通过[**这里**](http://115.28.152.100:8000)访问我布署在阿里云上面的ajme项目。
### **5. 参考**
>[**LEMP Installation (Nginx, MariaDB, PHP And phpMyAdmin) on Ubuntu 14.04 in 4 steps**](http://valuebound.com/resources/blog/lemp-installation-nginx-mariadb-php-and-phpmyadmin-ubuntu-1404-in-4-steps)
>[**Setting up Django and your web server with uWSGI and nginx**](http://uwsgi-docs.readthedocs.org/en/latest/tutorials/Django_and_nginx.html)
>[**阿里云部署 Flask + WSGI + Nginx 详解**](http://www.cnblogs.com/Ray-liang/p/4173923.html)