---
title: django
date: 2022-07-16 15:23:53
tags: django
---

# django

## 1.安装django

> 会在pythonXX的Scripts目录中生成 django-admin.exe【用于创建django项目中的文件和文件夹】

```python
pip install django
```

## 2.创建项目

> django中项目会有一些默认的文件和默认文件夹

### 2.1在终端创建

```shell
"C:\Python310\Scripts\django-admin.exe" startproject 项目名称
```

```shell
# 如果C:\Python310\Scripts已加入环境变量
django-admin startproject 项目名称
```

### 2.2Pycharm

<img src="django.assets/image-20220716155215425.png" alt="image-20220716155215425" style="zoom: 67%;" />

> pycharm创建django项目会比命令行多生成template目录，以及在setting.py文件中做了修改

<img src="django.assets/image-20220716155907733.png" alt="image-20220716155907733" style="zoom: 67%;" />

默认项目的文件介绍

```
pycharmDjango
	manage.py 		【项目的管理，启动项目、创建app、数据管理】【常用，默认不动】
	pycharmDjango
		__init__.py 
		setting.py	【项目配置】【经常修改】
		urls.py		【URL和函数的对应关系】【经常修改】
		asgi.py		【接收网络请求】
		wsgi.py		【接收网络请求】
```



## 3.创建app

<img src="django.assets/image-20220716161522565.png" alt="image-20220716161522565" style="zoom:67%;" />



## 4.基本顺序

- 注册app【settings.py】

<img src="django.assets/image-20220716162523437.png" alt="image-20220716162523437" style="zoom:67%;" />

- 编写URL和视图函数对应关系【urls.py】

<img src="django.assets/image-20220716162803315.png" alt="image-20220716162803315" style="zoom:67%;" />

- 编写视图函数【app01/views.py】

<img src="django.assets/image-20220716163027748.png" alt="image-20220716163027748" style="zoom:67%;" />

- 启动django项目

  - 命令行启动

  ```shell
  python manage.py runserver
  ```

  - 直接在pycharm点击运行

<img src="django.assets/image-20220716163414316.png" alt="image-20220716163414316" style="zoom:67%;" />

- 访问

<img src="django.assets/image-20220716163739004.png" alt="image-20220716163739004" style="zoom:67%;" />

- templates模板

<img src="django.assets/image-20220716164853289.png" alt="image-20220716164853289" style="zoom:67%;" />



- 静态文件

> 放在app目录下的static目录中

<img src="django.assets/image-20220716170350323.png" alt="image-20220716170350323" style="zoom:67%;" />