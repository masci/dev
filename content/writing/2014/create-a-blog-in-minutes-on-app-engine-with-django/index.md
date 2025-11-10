---
title: "Create a blog in minutes on App Engine with Django and Cloud Sql"
description: ""
category:
tags: ["django", "google", "python", "appengine", "cloudsql"]
date: "2014-02-10"
url: "create-a-blog-in-minutes-on-app-engine-with-django"
---

## Intro

Django was actively supported at an early stage of the Python runtime in App Engine SDK through the notable
[django-nonrel](http://www.django-nonrel.org) framework, a fork of the original project that adds support for NoSql databases.
But starting from the App Engine SDK 1.6.2, released more than two years ago, you can instead deploy Django's official releases
and take advantages from the whole stack using Google Cloud Sql.

## Case study

We're going to setup a minimal project using [Zinnia](http://django-blog-zinnia.com/), a blog engine built on top of Django and
a fairly complex web application that leverages several components of the framework, a good benchmark for showing how easy can
be deploying on App Engine.

## Prerequisites

Setting up the Google Cloud services goes beyond the scope of this article and is [well documented](https://cloud.google.com/developers/),
as well as having a working Python environment,  so the following it's assumed:

* you already started a Google Cloud project
* a Google Cloud Sql instance is up and running and you created a database for this project
* you created a bucket on Google Cloud Storage to store media files
* you have a working installation of Python 2.7 and pip on your local machine
* you installed and configured the Python App Engine SDK on your local machine

For the last point, make sure that issuing import google from a Python prompt does not raise any error.
Even if not required, I strongly recommend to use [virtualenv](http://www.virtualenv.org/en/latest/) to isolate the
Python environment for this project.

## Bootstrap

Let's start installing Django. The latest version available in the App Engine
[Python 2.7 environment](https://developers.google.com/appengine/docs/python/tools/libraries27) is the 1.5, so we go for the same:

    pip install django<1.6

Once finished, we can start an empty project:

    django-admin.py startproject myblog

This will create the typical Django application layout:

    myblog
     |_ myblog
     |_ manage.py

The project needs some dependencies that can be listed in a plain text file, one package per line, so that pip can install them all at once.
Along with the package name we can specify the version number, so that requirements won't change across different installations. Let's put the
following in a file called requirements.txt and save it at the root of the project:

    django-blog-zinnia==0.13
    django-appengine-toolkit
    pillow

Then we install the dependencies with:

    pip install -r requirements.txt

After pip finished we can finally start coding.

## Configure Django and Zinnia

First of all, we need to tell Django which application we want to use in our project, so open myblog/myblog/settings.py file and add these lines
to `INSTALLED_APP` setting:

    INSTALLED_APPS = (
        # other stuff here,
        'django.contrib.admin',
        'django.contrib.comments',
        'tagging',
        'mptt',
        'zinnia',
        'appengine_toolkit',
    )

the last application, [appengine_toolkit](https://github.com/masci/django-appengine-toolkit), is an helper that will make easier accessing some
features of App Engine from a Django project, we will see how in a moment.
We want to put all the static files (javascripts, css, images) in a folder called static at the root of our project (to be clear, along with the
`manage.py` module). Django can automatically collect such files if we set the variable STATIC_ROOT in settings.py with the full path to the
desired folder. We want to build an absolute path that will work both in local and production environments, so it can be convenient to add a
variable `BASE_DIR` to the `settings.py` pointing to the project root in a portable manner:

    BASE_DIR = os.path.dirname(os.path.dirname(__file__))

We can then refer the absolute path to the static folder as follows:

    STATIC_ROOT = os.path.join(BASE_DIR, 'static')

Zinnia uses a template context we need to set along with Django's default contexts so we add this block of code in `settings.py` module:

    TEMPLATE_CONTEXT_PROCESSORS = (
        'django.contrib.auth.context_processors.auth',
        'django.core.context_processors.i18n',
        'django.core.context_processors.request',
        'django.core.context_processors.media',
        'zinnia.context_processors.version',
    )

Following lines must be added to our project's urls.py in order to display the blog:
from django.contrib import admin

    admin.autodiscover()
        urlpatterns = patterns('',
        url(r'^admin/', include(admin.site.urls)),
        url(r'^weblog/', include('zinnia.urls')),
        url(r'^comments/', include('django.contrib.comments.urls')),
    )

## Configure App Engine

Now we need to create the yaml file containing App Engine application settings. At the root of the project create an app.yaml text file
containing the following:

    application: your_project_id_here
    version: 1
    runtime: python27
    api_version: 1
    threadsafe: true

    libraries:
    - name: django
      version: "1.5"
    - name: PIL
      version: "1.1.7"
    - name: MySQLdb
      version: "latest"

    builtins:
    - django_wsgi: on

    env_variables:
      DJANGO_SETTINGS_MODULE: 'myblog.settings'
      DATABASE_URL: 'mysql://root@your-project-id:sql-instance-name/database-name'

    handlers:
    - url: /static
      static_dir: static

Some parameters need to be adjusted with actual data, in particular we have to provide our Google Cloud project ID and the Cloud SQL instance
name.

## Configure database

The `DATABASE_URL` environment variable contains all the parameters needed to perform a connection from an App Engine application to our
database. Just add the following code to the settings.py to make Django capable to parse and make use of such parameters:

    import appengine_toolkit
    DATABASES = {
        'default': appengine_toolkit.config(),
    }

    APPENGINE_TOOLKIT = {
        'APP_YAML': os.path.join(BASE_DIR, 'app.yaml'),
    }

That's all and from now on, all we have to do for changing database connection parameters is to modify the `DATABASE_URL` environment variable
and deploy the application again.

## File storaging

We will store uploaded files in a bucket on Google Cloud Storage and we will let Django handle the upload process and then ask the Blobstore API
for a link to statically serve the same files. All we need to do is telling Django the bucket name and the Python class to use to talk to Cloud
Storage API:

    APPENGINE_TOOLKIT = {
        # other settings here
        'BUCKET_NAME': 'zinnia-uploads',
    }
    DEFAULT_FILE_STORAGE = 'appengine_toolkit.storage.GoogleCloudStorage'
    STATICFILE_STORAGE = 'appengine_toolkit.storage.GoogleCloudStorage'

## Deploy

Configuration steps are over, time to create the database schema with Django's built in management commands. Before proceeding, we have to set
the `DATABASE_URL` environment variable on the local machine performing the command. This is because we need to connect to the Cloud SQL
instance from the local machine and the connection string is slightly different from the one you would use in production, notice the `rdbms://`
component:

    export DATABASE_URL='rdbms://root@your-project-id:sql-instance-name/database-name'

With the variable set, issue the following command:

    python manage.py syncdb

During the schema creation we will prompted for username and password to assign to the admin user.
Now we need to provide application dependencies and App Engine has a peculiar approach to this: it requires that every piece of software which
is not already provided by the [Python Environment](https://developers.google.com/appengine/docs/python/tools/libraries27) has to be uploaded
together with application code during the deployment process. Instead of
mangling our local Python environment we will use a functionality provided by django_appengine_toolkit package. It adds a management command to
Django that symlinks all the dependencies needed in a folder inside the project root, making that folder available to the Python environment. We
issue the command:

    python manage.py collectdeps -r requirements.txt

and if everything is fine we will have a libs directory inside the project root containing all the dependencies needed.
Now we need to collect all the static files in one place, that's the static directory at the project root. Just issue the command:
python manage.py collectstatic
and we should find a folder named static at the project root that contains all the files needed by our application.
Now the final step, the actual deployment. If we are on a Mac we can use the Google App Engine Launcher tool and complete the deployment through
a graphical interface. Otherwise on Linux just issue this command in our project root:

    appcfg.py --oauth2 update .

Check out for any error and try accessing your application with a browser, you should see the Zinnia home page.
You can find the code of the example application in [my repo on GitHub](https://github.com/masci/django_appengine_example).

## Conclusions

These days App Engine seems to be a land the Django community forgot, but I think times are good for a change: the brand new Cloud Console and
the gcloud Tool, new services like Cloud Sql and the efforts in supporting the Python SDK can make the life of a Djangonaut a lot easier on the
Google platform. Sure, documentation should improve as well as the support to some client libraries but I think it's worth it and with a little
code we can get very close to something like "one click deploy".
