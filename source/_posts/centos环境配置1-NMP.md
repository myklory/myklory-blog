title: centos环境配置1--NMP
date: 2016-07-25 16:00:00
tags: [CentOS, Nginx, PHP, Mariadb]
---
在阿里云上面购买了ECS云服务器，安装了CentOS7.2 64位操作系统，为了达到可用的状态，还需要一经过一定的配置。
<!--more-->
#### **1. 创建一般用户**
阿里云上CentOS默认用户只有root，这会带来很大的安全隐患，需要添加一个普通用户来进行平常的操作。
下面使用useradd命令来创建一个用户：
```shell
useradd namine
passwd namine
```
执行这两个命令后会创建namine用户，我们平常使用就可以用这个用户，现在使用su命令切换到namine用户。
```shell
su namine
```
可以使用id namine命令来查看这个用户的一些信息。
但是现在我们使用sudo命令会提示namine is not in the sudoers file.  This incident will be reported.这说明namine用户还不能使用sudo，我们需要把这个用户加入sudoers文件中。和用户相关的文件有下面这些：
```shell
/etc/passwd 　　      //用户账户信息，可以看出用户名称
/etc/sudoers         //可以使用sudo命令的用户
/etc/shadow          //用户账户加密后信息，包括但不限于/etc/passwd中的信息
/etc/group           //组账户信息，可以看出组名称
/etc/gshadow   　　　 //组账户安全信息，包括但不限于/etc/group中的信息
/etc/default/useradd //账户创建时默认值
/etc/skel/           //包含默认文件的目录,具体作用尚不清楚
/etc/login.defs      //安全性的默认配置，与上面/etc/default/useradd有区别
```
由于/etc/sudoers文件默认权限是-r--------， 只有root可读，要先设为root可写。在root环境下使用下面命令来设置写权限。
```shell
chmod u+w /etc/sudoers
```
chmod使用的u/g/o/a分别表示用户/组/其它用户/所有前三者， +/-表示增加或删除后面的权限。
然后使用root打开/etc/sudoers，在其中root ALL=(ALL) ALL添加namine用户，下面修改完成后的文件：
```shell
root ALL=(ALL) ALL
namine ALL=(ALL) ALL
```
修改完成后用su切换回namine用户，我们使用yum来安装一个软件来测试一下可不可以使用sudo命令，CentOS默认是没有安装vim的，我们就安装vim来进行测试。
```shell
sudo yum install vim
```
输入namine的密码后等一会vim就安装完成了。
到这里，就新加了一个一般用户到CentOS中，平常我们就使用这个用户来进行操作，不再使用root。
最后，useradd和adduser命令虽然都是添加一个新用户，但是他们的区别之一就是前者会生成用户主目录，后者不会生成用户主目录。

#### **2. 配置NMP环境**

LNMP环境包括Linux, Nginx, Mysql, PHP，我们已经安装了CentOS，下面就来安装配置其它三种软件。

##### **2.1 安装配置Nginx**

CentOS源中已经集成了Nginx1.6.3版本，我们要安装1.10.1，需要添加新源：

```shell
rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
```

然后使用yum命令安装。

```shell
sudo yum install nginx
```

安装完成之后就可以测试运行一下是否安装成功，打开nginx服务。

```shell
sudo systemctl start nginx.service
```

然后在浏览器中打开http://115.28.152.100 查看能不能访问nginx欢迎页。很遗憾，提示bad geteway。查看一下CentOS的防火墙是不是打开。

```shell
sudo systemctl status firewalld
```

防火墙是打开的，我们要将nginx加入防火墙的例外。

```shell
sudo firewall-cmd --add-service=http
```

然后重新加载防火墙配置。

```shell
sudo systemctl reload firewalld
```
现在再次访问[http://115.28.152.100](http://115.28.152.100) 可以看到浏览器显示出了nginx的欢迎页。也可以使用绑定到此IP上的域名来访问[http://www.myklory.com](http://www.myklory.com)

还需要将nginx设置为随系统启动。

```shell
sudo systemctl enable nginx.service
```

至此，nginx已经安装完成。


##### **2.2 安装MariaDB**

直接使用yum命令安装MariaDB即可。

```shell
sudo yum install mariadb-server
```

MariaDB默认的root密码是空。需要重置一下root密码，运行下面的命令进行重置。

```shell
sudo mysql_secure_installation
```

利用花密生成新的密码，其它直接默认即可。

同样，开始运行服务和加入开机启动。

```shell
sudo systemctl start mariadb
sudo systemctl enable mariadb
```

##### **2.3 安装php**

这里我们安装默认源的5.4即可。安装php, php-fpm, php-mbstring, php-mysql四个软件包。

```shell
sudo yum install php php-fpm php-mbstring php-mysql
```

完成之后进行配置， 打开/etc/php-fpm.d/www.conf 文件配置php-fpm，我们通过php-fpm来解释nginx发过来的php请求。

```shell
sudo vim /etc/php-fpm.d/www.conf
```

作出如下修改：

```shell
listen.owner = nobody
listen.group = nobody
user = nginx
group = nginx
```

如果需要使用unix socket和nginx通讯则根据注释修改 listen = 127.0.0.1:9000这一行。

开始运行php-fpm服务

```shell
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
```

配置nginx，支持php。

打开nginx默认的配置文件/etc/nginx/conf.d/default.conf。

作出如下修改

```ini
 server {
     listen       80;
     server_name  localhost;

     #charset koi8-r;
     #access_log  /var/log/nginx/log/host.access.log  main;
     root   /usr/share/nginx/html;

     location / {
         index  index.php index.html index.htm;
     }

     #error_page  404              /404.html;

     # redirect server error pages to the static page /50x.html
     #
     error_page   500 502 503 504  /50x.html;
     location = /50x.html {
         root   /usr/share/nginx/html;
     }

     # proxy the PHP scripts to FastCGI server listening on 127.0.0.1:9000
     #
     location ~ \.php$ {
     #   root           html;
         fastcgi_pass   127.0.0.1:9000;
         fastcgi_index  index.php;
     #   fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
         fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
         include        fastcgi_params;
     }

     # deny access to .htaccess files, if Apache's document root
     # concurs with nginx's one
     #
     #location ~ /\.ht {
     #    deny  all;
     #}
}
```

测试能不能访问php网页， 在上面配置文件中的root所指向的路径下新建phpinfo.php文件，加入以下内容。

```php
<?php phpinfo(); ?>
```

访问[http://www.myklory.com/phpinfo.php](http://www.myklory.com/phpinfo.php) 可以正常显示php的安装信息，说明php+nginx已经安装成功。

##### **2.4 安装phpMyAdmin**

最后来配置mariadb管理工具phpMyAdmin， 先安装：

```shell
sudo yum install phpmyadmin
```

安装完成后将phpMyAdmin连接到nginx目录中。

```shell
sudo ln -s /usr/share/phpMyAdmin/ /usr/share/nginx/html/phpMyAdmin/
```

完成之后在浏览器中访问[http://www.myklory.com/phpMyAdmin](http://www.myklory.com/phpMyAdmin) 打开网页后看能不能登陆，能则说明安装成功。
