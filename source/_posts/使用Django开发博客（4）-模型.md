title: 使用Django开发博客（4）--模型
date: 2015-09-23 17:05:44
tags: [Django]
---
这一章我们创建博客的模型。
<!--more-->
### **使用git创建分支**
为了方便开发，我们每一章都使用不同的分支进行，不采用tag方式。
```bash
git checkout -b algblog4
```
使用下面命令可以查看本地和远程分支情况，前面带*号的表示当前正在使用的分支。
```bash
git branch -a
```
下面我们将在分支algblog4上面进行本章的开发工作。
### **设置数据库**
我采用的是mysql数据库，如果采用默认的sqlite3数据库可以跳过此步骤。
打开amje/setting.py文件，在DATABASES配置项中作如下修改：
```python
DATABASES = {
    'default':{
        'ENGINE':'django.db.backends.mysql', #数据库引擎
        'NAME':'algblog', #数据库名称
        'USER':'root', #用户名
        'PASSWORD':'root', #密码
        'HOST':'', #数据库位置，本机留空
        'PORT':'', #数据库端口，默认留空
    }
```
现在Django会使用MySQL数据库引擎。
接下来在MySQL中建立新的‘algblog’数据库，具体过程不在这里展开。
在我的项目里，使用python自带的mysql库不能正常操作mysql，所以使用了pymysql库。使用pip安装
```bash
pip install pymysql
```
在amje/\__init__.py中增加下列代码：
```python
import pymysql
pymysql.install_as_MySQLdb()
```
这样，Django就可以正常访问MySQL数据库了。
### **创建模型**
对于一个简单的博客来说，我们需要分类和文章两个模型。
#### **建立模型**
打开algblog/models.py文件，添加三个模型
```python
from django.db import models
from django.conf import settings

#文章的状态
STATUS = {
	0: '正常',
	1: '草稿',
	2: '删除',
}

# Create your models here.
class Category(models.Model):
	"""docstring for Category"""
	name = models.CharField('名称', max_length = 40)
	parent = models. ForeignKey('self', default = None, blank = True, null = True, verbose_name='上级分类')
	rank = models.IntegerField('排序',  default = 0)
	status = models.IntegerField('状态', default = 0, choices = STATUS.items())

	create_time = models.DateTimeField('创建时间', auto_now_add = True)

	class Meta:
		verbose_name_plural = verbose_name = '分类'
		ordering = ['rank', '-create_time'] #按时间排序

	def __str__(self):
		if self.parent:
			return '%s --> %s' % (self.parent, self.name)
		else:
			return self.name
 class Tags(models.Model):
     name = models.CharField(max_length = 100, verbose_name = '标签名称')
     create_time = models.DateTimeField('创建时间', auto_now = True)

     class Meta:
         verbose_name = verbose_name_plural = '标签'

     def __str__(self):
         return self.name

class Article(models.Model):
	"""docstring for Article"""
	author = models.ForeignKey(settings.AUTH_USER_MODEL, verbose_name = '作者')
	category = models.ForeignKey(Category, verbose_name = '分类')
	title = models.CharField(max_length = 100, verbose_name='标题')
	tags = models.ManyToManyField(Tags, verbose_name='标签')
	content = models.TextField(verbose_name = '正文')

	view_times = models.IntegerField(default = 0, verbose_name='浏览次数')
	agree_times = models.IntegerField(default = 0, verbose_name='被赞次数')
	is_top = models.BooleanField(default = False, verbose_name = '置顶')
	rank = models.IntegerField(default = 0, verbose_name='排序')
	status = models.IntegerField(default = 0, choices = STATUS.items(), verbose_name ='状态')

	pub_time = models.DateTimeField(default = False, verbose_name = ' 发布时间')
	create_time = models.DateTimeField('创建时间', auto_now_add = True)
	update_time = models.DateTimeField('发布时间', auto_now = True)

    #返回tags列表
	def get_tags(self):
		return self.tags.split(',')

	class Meta:
		verbose_name = verbose_name_plural = '文章'
		ordering = ['rank', '-is_top', '-pub_time', '-create_time'] #排序规则

	def __str__(self):
		return self.title
```
其中\__str__(self) 函数的作用是告诉Article对象要怎么表示自己, 一般系统默认使用<Article: Article object> 来表示对象, 通过这个函数可以告诉系统使用title字段来表示这个对象。
有关模型的详细信息可以看[**这里**](https://docs.djangoproject.com/en/1.8/topics/db/models/)。
### **迁移**
模型建立好之后，需要迁移至数据库，使用下面命令生成迁移文件
```bash
python manage.py makemigrations
```
我在使用这个命令的时候提示No changes detected，最后发现是没有把algblog加入配置文件，打开amje/setting.py文件，在INSTALLED_APPS项最后加入algblog：
```python
INSATLL_APPS = (
    ...
    'algblog',
)
```
现在再运行上面命令，可以发现在algblog/migrations/目录下生成了0001_initial.py文件，这就是我们的数据迁移文件。可以使用下面命令查看此文件生成的SQL语句：
```bash
python manage.py sqlmigrate algblog 0001
```
最后，使用下面命令正式迁移到数据库
```bash
python manage.py migrate
```
migrate后面可跟具体的APP名字，只迁移对应的APP。
### **提交到版本库**
关于模型就创建完了，最后需要把我们的修改合并到master并提交到版本库。
```bash
git add . #添加所有修改文件到版本库
git commit -m "创建Category,Article模型，增加mysql数据库支持"
git push --set-upstream origin algblog4 #推送分支algblog4到远程版本库，现在打开github就会看见此分支
git checkout master #切换到master分支
git merge algblog4 #将分支algblog4合并到master分支
git push #提交master分支到远程库
```
