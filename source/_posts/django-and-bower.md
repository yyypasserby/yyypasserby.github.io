---
title: Practices on Django & Bower
date: 2014-12-18 11:22:33
categories:
  - web
tags:
  - django
  - bower
thumbnail: /img/django-and-bower/django.png
---

As a front-end developer, most of you may know [bower](http://bower.io) these days. Bower is a powerful tools to make your front-end more modular and easy to manage.

And [Django](https://www.djangoproject.com/) is also very popular in the back-end deveopment. [Douban](http://www.douban.com/), [Zhihu](http://www.zhihu.com/) all uses Django as their backend framework.

Today, I am just going to talk about something I concluded from my project in cooperate Django and bower and they really make my website dev much more convenient.

## Virtualenv
This tool is always used in many python project. Python is undoubtedly a great language with diversity, but it also encounter many problems as its diversity. We may frequently find some package conflicts with each other, that is because the package is design by different people, they won't communicate with each other until facing conflict. So usually we use the [virtualenv](https://virtualenv.pypa.io/) to avoid these uncompatible problems as much as we can. After you set up your [setuptools](https://pypi.python.org/pypi/setuptools) and [pip](https://pypi.python.org/pypi/pip/), you can easily get virtualenv by

```bash
pip install virtualenv
```

After installing virtualenv, you can set up a individual virtual environment for your project.

```bash
virtualenv env
source env/bin/activate
```

And all the things you doing after is just change the environment in a virtual way, it won't conflict with your other packages if they are not in the same environment. To my own point of view, I will start up a brand new environment for a new project even some of its dependency overlaps with my project before. You could also use the same virtual env when doing all your Django project. It's all up to you.

## Django
You can install Django in your virtual environment now. 

```bash
pip install django
```

The lastest version of Django is 1.7, and its has a great update in 1.7 about the model operation. It has merge the function of [south](http://south.aeracode.org/) project into it and make the modification of the models more convenient. This feature still has some improvements. The process is a little bit different when we build our tables. In the former versions, we do something like following

```python
# app/models.py
class Books(models.Model):
    name = models.TextFields()

# project/settings.py
INSTALLED_APPS = (
    ...
    'app',
)
```

And do these in the command

```bash
python manage.py syncdb
```

And all your models will sync to your databases.
**But now in 1.7**, we will have something more to do, **instead of the command above.**

```bash
python manage.py makemigrations app
```

First we should make the migrations of the app and records them in the `migration` folder. The structure is just like the following

```
migrations
├── 0001_initial.py
├── 0001_initial.pyc
├── __init__.py
└── __init__.pyc
```

The file produced by this step is to make a image of this version and Django could identify the future changes by comparison with this image.

```bash
python manage.py migrate app
```

The operation is just like the `syncdb` in the former version. The models are created in the databases.

If you are using [sqlite3](http://sqlite.org/), you can use `.tables` command to show all the tables in your databases.

## Bower
To my personal custom, in some small projects, I would like to make my utils static files that used in all apps in a folder in the root level. Just like

```bash
├── db.sqlite3
├── app
├── manage.py
├── static
├── template
└── project
```

This static folder is used to store bower repository for me.

```bash
cd static
bower init
```

Then you can install any front-end framework as you want. To me, these three framework is necessary,

```
bower install requirejs --save
bower install bootstrap --save
bower install jquery --save
```

After installed these packages, we could use either `<script>` ways or `rejuirejs` ways to include them in our django templates. I'm just going to show you the basic way.

Django has several ways to deal with static files. `STATIC_ROOT` and `STATICFILES_DIRS` are two stantard methods. When we use static file, we must ensure an app is activated in `INSTALLED_APPS`.

```python
INSTALLED_APPS = (
    'django.contrib.staticfiles',
)
```

I prefer to store all my dependency in a global static folder, so I just need to add

```python
# project/settings.py
STATICFILES_DIRS = (
    os.path.join(BASE_DIR, 'static'),
)
```

```html
<!--templates/index.html-->
<head>
    {% raw %}{% load staticfiles %}{% endraw %}
    {% raw %}<link href="{% static "bower_components/path/to/staticfiles" %}">{% endraw %}
</head>
```

Then the engine could find these static files for me.

