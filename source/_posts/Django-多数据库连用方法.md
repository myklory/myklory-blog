title: Django 多数据库连用方法
date: 2015-09-01 15:46:00
tags: [Django]
---
在Django[**文档**](https://docs.djangoproject.com/en/1.8/topics/db/multi-db/)中详细描述了使用多数据库的方法。这里简单记录一下使用方法,其中amje是项目，algblog是APP。
<!--more-->
### 增加多数据库连接配置
打开Django项目的setting.py文件，在DATABASE配置项增加新的数据库连接配置
```python
DATABASES = {
    'default':{
        'ENGINE':'django.db.backends.mysql', #数据库引擎
        'NAME':'test', #数据库名称
        'USER':'root', #用户名
        'PASSWORD':'root', #密码
        'HOST':'', #数据库位置，本机留空
        'PORT':'', #数据库端口，默认留空
    }
    'algblog':{
        'ENGINE':'django.db.backends.mysql', #数据库引擎
        'NAME':'algblog', #数据库名称
        'USER':'root', #用户名
        'PASSWORD':'root', #密码
        'HOST':'', #数据库位置，本机留空
        'PORT':'', #数据库端口，默认留空
    }

#设置数据库路由路径，让Django知道使用哪个类来路由数据库
DATABASE_ROUTER = ['amje.database_router.DatabaseRouter']

#设置路由字典，在路由类中使用
DATABASE_MAP = {
    'algblog':algblog', #algblog项目使用algblog数据库，其他使用default数据库配置
}
```

### 多数据路由文件
在Django项目目录中增加文件database_router.py
```python
from amje.settings import *

class DatabaseRouter(object):
	"""docstring for DatabaseRouter"""
	def db_for_read(self, model, **hints):
		if model._meta.app_label in DATABASES:
			return DATABASE_MAP.get(model._meta.app_label)
		return None

	def db_for_write(self, model, **hints):
		if model._meta.app_label in DATABASES:
			return DATABASE_MAP.get(model._meta.app_label)
		return None

	def allow_relation(self, obj1, obj2, **hints):
		if  obj1._meta.app_label ==DATABASE_MAP.get(obj1._meta.app_label) or \
		obj2._meta.app_label ==DATABASE_MAP.get(obj2._meta.app_label)  :
			return True
		return None

	def allow_migrate(self, db, app_label, model =None, **hints):
		if db in DATABASE_MAP:
			return DATABASE_MAP.get(app_label) == db
		elif app_label in DATABASE_MAP:
			return False
		return None
```
### 迁移模型到数据库
迁移数据库时需要手动指定项目和数据库配置。
生成迁移文件：
```bash
python manager.py makemigrations algblog
```
迁移数据库：
```bash
python manager.py migrate algblog --database=algblog
```
```python
import pymysql
pymysql.install_as_MySQLdb()
```
