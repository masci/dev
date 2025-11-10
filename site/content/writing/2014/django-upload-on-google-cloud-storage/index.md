---
title: "Uploading files to Google Cloud Storage with Django"
description: ""
category:
tags: ["django", "google", "python", "appengine", "gcs"]
date: "2014-05-02"
url: "django-upload-on-google-cloud-storage"
---

## Intro

On of the features of [Django Appengine Toolkit](https://github.com/masci/django-appengine-toolkit) is simplifying
the work needed to configure Google Cloud Storage as a static files storage for Django applications running on
Google App Engine. Infact all you have to do is writing something like this in your settings.py module:

```py
    APPENGINE_TOOLKIT = {
        'APP_YAML': os.path.join(BASE_DIR, 'app.yaml'),
        'BUCKET_NAME': 'media-uploads',
    }
    DEFAULT_FILE_STORAGE = 'appengine_toolkit.storage.GoogleCloudStorage'
    STATICFILE_STORAGE = 'appengine_toolkit.storage.GoogleCloudStorage'
```

## A complete example

[This repo](https://github.com/masci/django_cloudstorage_example) contains a minimalistic Django project
implementing a file storage application that lets users upload, listing, retrieve and delete files. The project has just
one app implementing all the logic, defining the model and exposing the views. For detailed instructions on how to
setup a Django project on App Engine with `django-appengine-toolkit` please check out
[this blog post](http://dev.pippi.im/2014/02/10/create-a-blog-in-minutes-on-app-engine-with-django/).
Now let's take a look at the code.

### The Model

```py
    class Document(models.Model):
        docfile = models.FileField(upload_to='documents/%Y/%m/%d')

        def delete(self, *args, **kwargs):
            storage, path = self.docfile.storage, self.docfile.path
            super(Document, self).delete(*args, **kwargs)
            storage.delete(path)
```

Pretty easy, we have just one field containing the file. Notice the delete method we're going to use so that
when an instance is deleted, the same will happen to corresponding file on Cloud Storage.

### The views

Hail to the Class Based Views! Look at how few lines of code we need for the main view, implementing the listing and
the logic for the uploads, form included:

```py
    class FileManagerView(CreateView):
        model = Document
        success_url = reverse_lazy('main')

        def get_context_data(self, **kwargs):
            kwargs['object_list'] = Document.objects.all()
            kwargs['fava'] = 'rava'
            return super(FileManagerView, self).get_context_data(**kwargs)
```

Since we need to show the list of files **and** the form to upload them on the same page, we cannot use a `CreateView` as is,
what we need is a `CreateView` and `ListView` hybrid instead, thus the hack of overriding `get_context_data`: we inject the queryset
in the context so the template can render properly.

The relevant html code in the template looks like this:

```html
    <ul>
    {% for object in object_list %}
      <li>
        <form action="{% url 'delete' object.id %}" method="post">{% csrf_token %}
          <a href="{{ object.docfile.url }}">{{ object.docfile.name }}</a>
          <input type="submit" value="Delete" />
        </form>
      </li>
    {% empty %}
      <li>No documents.</li>
    {% endfor %}
    </ul>

    <form  enctype="multipart/form-data" action="" method="post">{% csrf_token %}
        {{ form.as_p }}
        <input type="submit" value="Upload" />
    </form>
```

Notice we render a form for each file listed, so we can make a `POST` request directly, without passing for a confirmation view
as usual when using `DeleteView` generics. Let's see the View code:

```py
    class FileRemoveView(DeleteView):
        model = Document
        success_url = reverse_lazy('main')
```

Ok, this was short. Basically we only need to tell to the class based view which is the model and where to go once the istance
is deleted. Wow.

### The urls

Quick and dirty: mount the two views to the urls:

```py
    urlpatterns = patterns('',
        url(r'^$', FileManagerView.as_view(), name='main'),
        url(r'^delete/(?P<pk>\d+)/$', FileRemoveView.as_view(), name='delete'),
    )
```

That's all, have fun deploying on App Engine!
