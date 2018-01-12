---
title: "django静态文件配置"
author: Ringo Yungpo Kao
date: January 02, 2018
output:
  word_document:
    path: /Exports/notes/djnagodjango静态文件配置模板引擎.docx
---
# djnagodjango静态文件配置模板引擎
静态文件指像css,js,images之类的文件，在Django里面静态文件的处理与一般的视图是不一样，新手往往容易犯迷糊，本文做一下总结：

## 简介

开发网站的过程中，我们需要使用到大量的静态文件，例如js，CSS，图片等等，一般我们会在网页中以URL的形式引用它们。Django提供了一些特殊的机制来处理静态文件。默认情况下（如果没有修改STATICFILES_FINDERS的话），Django首先会在STATICFILES_DIRS配置的文件夹中寻找静态文件，然后再从每个app的static子目录下查找，并且返回找到的第一个文件。所以我们可以将全局的静态文件放在STATICFILES_DIRS配置的目录中，将app独有的静态文件放在app的static子目录中。存放的时候按类别存放在static目录的子目录下，如图片都放在images文件夹中，所有的CSS都放在css文件夹中，所有的js文件都放在js文件夹中。

## 概述

静态文件交由Web服务器处理，Django本身不处理静态文件。简单的处理逻辑如下(以nginx为例)：

               URI请求-----> 按照Web服务器里面的配置规则先处理，以nginx为例，主要求配置在nginx.conf里的location

                         |---------->如果是静态文件，则由nginx直接处理

                         |---------->如果不是则交由Django处理，Django根据urls.py里面的规则进行匹配

以上是部署到Web服务器后的处理方式，为了便于开发，Django提供了在开发环境的对静态文件的处理机制，方法是这样：

1. 在INSTALLED_APPS里面加入'django.contrib.staticfiles',
2. 在urls.py里面加入

```python
if settings.DEBUG:  
   urlpatterns += patterns('', url(r'^media/(?P<path>.*)$', 'django.views.static.serve', {'document_root': settings.MEDIA_ROOT }),   
        url(r'^static/(?P<path>.*)$','django.views.static.serve',{'document_root':settings.STATIC_ROOT}), )  
```

3. 这样就可以在开发阶段直接使用静态文件了。

## MEDIA_ROOT和MEDIA_URL

而静态文件的处理又包括STATIC和MEDIA两类，这往往容易混淆，在Django里面是这样定义的：

MEDIA:指用户上传的文件，比如在Model里面的FileFIeld，ImageField上传的文件。如果你定义

MEDIA_ROOT=c:\temp\media，那么File=models.FileField(upload_to="abc/")，上传的文件就会被保存到c:\temp\media\abc
举个例子：

```python
class blog(models.Model):  
       Title=models.charField(max_length=64)  
       Photo=models.ImageField(upload_to="photo")  
```

上传的图片就上传到c:\temp\media\photo，而在模板中要显示该文件，则在这样写{{MEDIA_URL}}blog.Photo

在settings里面设置的MEDIA_ROOT必须是本地路径的绝对路径，一般是这样写

```python
PROJECT_PATH = os.path.abspath(os.path.dirname(__file__))  
MEDIA_ROOT=os.path.join(PROJECT_PATH,'media/').replace('\\','/')  
```

MEDIA_URL是指从浏览器访问时的地址前缀，举个例子：

```python
MEDIA_ROOT=c:\temp\media\photo  
MEDIA_URL="/data/"                     #可以随便设置  
```

在开发阶段,media的处理由django处理：

访问http://localhost/data/abc/a.png就是访问c:\temp\media\photo\abc\a.png

在模板里面这样写<img src="{{MEDIA_URL}}abc/a.png">

在部署阶段最大的不同在于你必须让web服务器来处理media文件，因此你必须在web服务器中配置，以便能让web服务器能访问media文件

以nginx为例，可以在nginx.conf里面这样：
```
     location ~/media/{
   root   /temp/
   break;
}
```
## STATIC_ROOT和STATIC_URL、

 STATIC主要指的是如css,js,images这样文件，在settings里面可以配置STATIC_ROOT和STATIC_URL,配置方式与MEDIA_ROOT是一样的，但是要注意

STATIC_ROOT与MEDIA_ROOT位置不能一样。

STATIC文件一般保存在以下位置：
1. STATIC_ROOT：在settings里面设置，一般用来放一些公共的js,css,images等。

2. app的static文件夹，在每个app所在文夹均可以建立一个static文件夹，然后当运行collectstatic时，Django会遍历INSTALL_APPS里面所有app的static文件夹，将里面所有的文件复制到STATIC_ROOT。因此，如果你要建立可复用的app，那么你要将该app所需要的静态文件放在static文件夹中。

 也就是说一个项目引用了很多app，那么这个项目所需要的css,images等静态文件是分散在各个app的static文件的，比较典型的是admin应用。

当你要发布时，需要将这些分散的static文件收集到一个地方就是STATIC_ROOT。


3. STATIC文件还可以配置STATICFILES_DIRS，指定额外的静态文件存储位置。


STATIC_URL的含义与MEDIA_URL类似。

## 小结
如果你发生在部署阶段找不到css,js，则可能是以下几个问题：

1. web服务器配置有问题，不同的部署方式对静态文件的处理有所不同。

2. 没有运行collectstatic将所需要的静态文件收集到STATIC_ROOT