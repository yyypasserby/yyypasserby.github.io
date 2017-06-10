---
title: Deploying Django 1.7 to Heroku
date: 2014-12-21 11:22:33
categories:
  - web
  - backend
tags:
  - django
  - deploy
---

![Django](/images/django-on-heroku/django.png)

[Django 1.7](https://www.djangoproject.com/) has been long released and its new features of migrations make our web development much easier when dealing with DBs.

[Heroku](https://www.heroku.com/) is a PaaS that managed by [git](http://www.git-scm.com/) and supports Django. If you are a man that don't want to spend any money on your website like me. I think Heroku is somewhat a good choice for you.

I used to deploy my website on [Openshift](https://www.openshift.com/) provided by redhat-cloud, however it sometimes was blocked by the GFW. The service of Openshift is great but considering the difficulty of maintainence, I gave up and turn to Heroku.

But all these PaaS has some several common nature:

- need to obey its permission on files, you could not easily customized your web app in some particular ways.

- need to obey the ways that settings given by the PaaS provider. You should install the local deploy tools to make your web app works find on your localhost. Sometimes if you don't obey the rules, your app won't work

- free service is often with slow reponse and unstablility.

For the reason above, the PaaS service is recommended to deploy some simple and regualr web app with limited functions, such as some frameworks demo, some homework presentation and etc.

## Prerequisites

The setups of Django project is demonstrated in this [blog]({filename}/2014-12-18-django-and-bower.md).

So after the creation of a project is done. Your project is maybe just like this.

```
hellodjango
├── hellodjango
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── wsgi.py
├── manage.py
└── venv
```

With the instruction given by the [Documentation](https://devcenter.heroku.com/articles/getting-started-with-django), we know that the Heroku platform need us to do some extra things to deploy your app.

**First**,  your need to install [Heroku-toolbelt](https://devcenter.heroku.com/articles/getting-started-with-python#set-up), this contains `heroku` command to operate and show logs of your project.

**Second**, the default database of Heroku is **Postgresql**, so your need to install [Postgres.app](http://postgresapp.com/) to your computer. I strongly recommend the Mac user to choose the app provided by the official website which could save your a lot of time.

**Thirdly**, especially for Django, you need to install the [django-toolbelt](https://pypi.python.org/pypi/django-toolbelt/0.0.1) to make your website work on Heroku in production state. I found the main thing that django-toolbelt has done is to deal with the static file, like css, js, image and etc. In [documentation](https://docs.djangoproject.com/en/1.7/) of Django, it is not recommended that to use staticfile serve provided by Django for some particular reasons including security. To install django-toolbelt easily, you can type the following command when you are in **virtualenv** mode

```bash
pip install django-toolbelt
```

If you encounter some problems like `could not find pg_config executable`, that means your Postgresql is not properly configured. You should add the `pg_config` into your environment variable. It is here if you use the **Postgres.app**

```
/Applications/Postgres.app/Contents/Versions/9.4/bin
```

## Configure Heroku

After you have done all these, we can start to configure something required by the Heroku to make your app running on the remote.

### Procfile
Procfile is used to configure the web container used to run **WSGI** application. The content is like

```
web: gunicorn <project_name>.wsgi --log-file -
```

Now you can type

```bash
foreman start
```

to check if your app is running well in the local environment.

### requirements.txt

This file is used for Heroku to install the dependency for your python web app. After you push your file up to the Heroku server, it will execute

```bash
pip install -r requirements.txt
```

to install the dependencies. So what we need to do is

```bash
pip freeze > requirements.txt
```

### Django settings

There are also something we need to modify in our Django project.

**settings.py**

```python
# Parse database configuration from $DATABASE_URL
import dj_database_url
DATABASES['default'] = dj_database_url.config()
# Honor the 'X-Forwarded-Proto' header for request.is_secure()
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')
# Allow all host headers
ALLOWED_HOSTS = ['*']
# Static asset configuration
import os
# Not essential because the settings.py in Django 1.7 has already 
# provided the definition of BASE_DIR and that seems to be more correct
# BASE_DIR = os.path.dirname(os.path.abspath(__file__))
STATIC_ROOT = 'staticfiles'
STATIC_URL = '/static/'
STATICFILES_DIRS = (
    os.path.join(BASE_DIR, 'static'),
)
```

**wsgi.py**

```python
from django.core.wsgi import get_wsgi_application
from dj_static import Cling

application = Cling(get_wsgi_application())
```

## Deploy to Heroku

Now your file structure is like

```
hellodjango
├── Procfile
├── hellodjango
├── manage.py
├── requirements.txt
└── venv
```

We know that Heroku use git to manage the file upload, so the first thing to do is to initiate a git repo and make all the things done committed to the repo.

```bash
git init
git add .
git commit -m'init commit'
```
The `.gitignore` file could contain these several lines.

```
venv/
*.pyc
staticfiles/
```

After the repo is set up, we shall create a heroku repo and add its url to our git remote repo. All these operation is encapsulated in one command.

```bash
heroku create
```

If you are the first time to use **heroku-toolbelt**, you had better login first

```bash
heroku login
```

Input your account and password according to the info.

Then you need to push your repo to the master branch of the remote repo in Heroku.

```bash
git push heroku master
```

Then, after Heroku make a the remote environment set up, you can now visit your website using `heroku open` and try to rename it using `heroku apps:rename <new_app_name>`.

There are also many useful commands to control your website, to see more detailed information, please see the [Documentation](https://devcenter.heroku.com/articles/getting-started-with-python).

## Suggestion

### Automatically config environment
The modification in **settings.py** may make your local debugging not performing right. So I added something more to make the changes just work in the remote.

1. Create a file naming `.env`, and type these

 ```
 ENV=LOCAL
 ```

2. Add one line before the modification and the final content shall be

 ```python
 if 'LOCAL' != os.environ.get('ENV', 'HEROKU'):
 # Parse database configuration from $DATABASE_URL
    import dj_database_url
    DATABASES['default'] = dj_database_url.config()
    ...
 ```

3. Config the remote environment variable, type the following command under your app root folder

 ```bash
 heroku config:set ENV=HEROKU
 ```

These operation just add an environment variable to the development env and production env and make the app itself to detect the environment changes.

### Use the Django admin

As we all know **Django admin** is a very useful module to give the developer a full view of his app. To activate this feature, we should do the following two things.

1. Configure the static files properly and include the related apps and middlewares in the corresponding settings.

2. Create a superuser of the remote app. Heroku provide us a way to execute instructions to the remote user. Just like,

 ```bash
 heroku run python manage.py createsuperuser
 ```

 Then it will be instruct you to create a superuser interactively like you just do locally.

 You know can visit the `<app_url>/admin` to operate your remote database happily.
