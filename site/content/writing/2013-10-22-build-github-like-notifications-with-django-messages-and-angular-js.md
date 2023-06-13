---
title: "Build GitHub like notifications with Django messages and AngularJS"
description: ""
category:
tags: ["django", "python", "angularjs"]
date: "2013-10-22"
slug: "build-github-like-notifications-with-django-messages-and-angular-js"
aliases:
 - "/2013/10/22/build-github-like-notifications-with-django-messages-and-angular-js/"
---

Foreword
--------

GitHub has a very nice notification system, very similar to a plain old email inbox. You receive a notification which
remains *unread* until you actually read it; then it's archived and removed from your *inbox*, which it happens could remain empty:

![github inbox](/images/github_notifications.png)

For those who don't know, Django ships a library for displaying "one-time" messages to the users, it's called _Message
Framework_ and you can find it in the `contrib` package. Messages are _produced_ during users' activity and delivered
subsequently; in the meantime, they are stored in cookies or sessions.

The ephemeral nature of Django's `contrib.messages` makes them not suitable for storing notifications in GitHub
style: notifications have to be persisted until user actually reads it, messages instead are marked as read the moment
they are, let's say, _observed_. Nevertheless Django message framework is flexible enough to let you provide your own
storage policy, and third-party applications like [Django Stored Messages](https://github.com/evonove/django-stored-messages)
use this feature to store messages in a persistent way (specifically, an sql database).

> Disclaimer: The frontend code will make use of AngularJs even if I am a total newbie and I don't really know how to
> angular. If you are a newbie too, please go read [this effective blog post](http://toddmotto.com/ultimate-guide-to-learning-angular-js-in-one-day/)
> by Todd Motto and then come back here. If you're not an Angular newbie, please take into account my code could offend
> you.

The application
---------------

Fire up a virtualenv and install Django:

    mkvirtualenv notification_example
    pip install django

Start an empty project:

    django-admin.py startproject notification_example

...and an app:

    cd notification_example
    python manage.py startapp notification

Now for some dependencies - install Django Stored Messages and [Django Rest Framework](http://http://django-rest-framework.org/):

    pip install django-stored-messages djangorestframework

Configure all the things! In `notification_example/settings.py` be sure to have these:

```py
    PROJECT_ROOT = os.path.join(os.path.abspath(os.path.dirname(__file__)), '..')

    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.sqlite3',
            'NAME': os.path.join(PROJECT_ROOT, 'db.sqlite'),
        }
    }

    INSTALLED_APPS = (
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.sites',
        'django.contrib.messages',
        'django.contrib.staticfiles',
        'notification',
        'stored_messages',
        'rest_framework',
    )

    MESSAGE_STORAGE = 'stored_messages.storage.PersistentStorage'
```

Now let's go for some views. We will provide a view to serve the homepage, plus a view to show messages for the current
logged in user.

The homepage view
-----------------

Django Stored Messages can persist messages only when they are sent to a valid user, and such user has to login for
viewing the messages, so we provide a login form directly inside the homepage. To produce some notifications, visiting the index will trigger a message as well. The code:

```py
    from django.views.generic import FormView
    from django.contrib.auth.forms import AuthenticationForm
    from django.contrib.messages import add_message
    from django.contrib.auth import login

    import stored_messages

    class IndexView(FormView):
        template_name = 'notification/homepage.html'
        form_class = AuthenticationForm
        success_url = '/'

        def get_context_data(self, **kwargs):
            add_message(self.request, stored_messages.STORED_INFO, 'You visited the homepage')
            return super(IndexView, self).get_context_data(**kwargs)

        def form_valid(self, form):
            login(self.request, form.get_user())
            return super(IndexView, self).form_valid(form)
```

We're going to use _class based views_, of course. Notice Django Stored Messages let us make use of the builtin
messages api, thus passing in a message of type `stored_messages.STORED_INFO` will cause that message to be stored on
the database.
The [homepage template](https://github.com/masci/notification_example/blob/master/templates/notification/homepage.html)
will be extended from a [basic Boostrap3 template](https://github.com/masci/notification_example/blob/master/templates/base.html), we're going onto details later.

The message view
----------------

This is a simple `TemplateView`, the only trick here is getting from the urlstring whether user wants to see all the
notifications or only the _unread_ ones:

```py
    from django.views.generic import TemplateView

    class MessagesView(TemplateView):
        template_name = 'notification/messages.html'

        def get(self, request, *args, **kwargs):
            if 'unread' in request.GET:  # quick and dirty
                kwargs['unread'] = True
            return super(MessagesView, self).get(request, *args, **kwargs)
```

The view code is rather simple because all the magic is left to Django Stored Messages template tags and its REST api.
The Html template for the message view will try to mimic GitHub's notification page,
[here is the code](https://github.com/masci/notification_example/blob/master/templates/notification/messages.html) and
this is the result:

![messages screenshot](/images/messages.png)

As you can see from this chunk:

```html
    <div class="col-md-9">
    {% if not unread %}
        {% stored_messages_archive 100 %}
    {% else %}
        ...
    {% endif %}
    </div>
```

If user requested the archive (i.e. to show messages that were already read), the template tag
`stored_messages_archive` provided by Django Stored Messages will show a list of `100` messages rendering
the template at `stored_messages/stored_messages_list.html`. Here is the template ovverrided to add Bootstrap3 classes:

```html
    {% if messages %}
        <ul class="list-group">
            {% for message in messages %}
                <li class="list-group-item">
                    {{ message.message }}
                </li>
            {% empty %}
                <li class="list-group-item">No messages here!</li>
            {% endfor %}
        </ul>
    {% endif %}
```

We will get into the details for the `else` branch in the messages template later.

Plug the urls
-------------

Nothing special here but notice the inclusion of the REST api urls coming from `stored_messages` package:

```py
    urlpatterns = patterns('',
        url(r'^logout/$', 'django.contrib.auth.views.logout',  {'next_page': '/'}, name='logout'),
        url(r'^$', IndexView.as_view(), name='home'),
        url(r'^messages/$', MessagesView.as_view(), name='messages'),
        url(r'^api/', include('stored_messages.urls')),
    )
```

Final touches
-------------

To add some noise to the notification stream, we will add messages for the user when she logs in and out:

```py
    def _user_logged_in(*args, **kwargs):
        add_message(kwargs['request'], stored_messages.STORED_INFO, 'You were logged in!')
    user_logged_in.connect(_user_logged_in)


    def _user_logged_out(*args, **kwargs):
        add_message(kwargs['request'], stored_messages.STORED_INFO, 'You were logged out!')
    user_logged_out.connect(_user_logged_out)
```

The Django app is complete now, time for some Javascript code!

The REST
--------

Even if Django Stored Messages has a template tag to show unread messages, for this demo we will use the REST api,
which let us retrieve unread messages and mark them as read. To interact with the api we use
[Angular](http://angularjs.org/), for the sake of simplicity we use a single controller:

```js
    var messageApp = angular.module('messageApp', []);

    messageApp.controller('MainCtrl', ['$scope', '$http', function ($scope, $http) {
        // Messages array
        $scope.messages = {};

        // ...
    }]);
```

Notice the injection of the `$http` object we will use to make http requests. The messages array will be filled with
data coming from the api, then it will be available through the `$scope` object. For the angular application to work
properly, the html code in our templates needs to be aware of the angular stuff - we do this in the `base.html` so that
every page could use angular facilities:

```html
    <!DOCTYPE html>
    <html ng-app="messageApp">
        <head>
        ...
```

and:

```html
    ...
    </head>

    <body ng-controller="MainCtrl">
        <!-- navbar -->
        <div class="navbar navbar-inverse navbar-fixed-top">
        ...
```

Inside the controller, this code will be added to retrieve all the unread messages for the logged-in user:

```js
    // ...

    // retrieve Messages from the restAPI
    $http({
        method: 'GET',
        url: '//127.0.0.1:8000/api/inbox/'
    })
    .success(function (data, status, headers, config) {
        $scope.messages = data;
    })
    .error(function (data, status, headers, config) {
        // something went wrong :(
    });

    // ...
```

If everything goes fine, `$scope.messages` will contain our messages and we can use them inside the DOM. To do this, we
need some angularities inside the html, for example in the `message.html` template:

```html
    {% verbatim %}
    <ul class="list-group" ng-if="messages.length">
      <li class="list-group-item" ng-repeat="message in messages">
        {{ message.message.date | date:'MMM d, y h:mm:ss a' }} - {{ message.message.message }}
          <a ng-click="markRead($index)" style="cursor:pointer">Mark as read</a>
      </li>
    </ul>
    {% endverbatim %}
```

The `ng-if` attribute determines if we have some messages to show. If we do have, the `ng-repeat` attribute will take
care of iterating the messages and show them in the DOM through angular's template tags.

> Warning: we're mixing Django and Angular templates there and since they share the same template syntax (this could
> be changed in angular but it's not generally advisable) we need to wrap angular code inside Django's `verbatim` tags.

In the html code above notice this:
```html
    <a ng-click="markRead($index)" style="cursor:pointer">Mark as read</a>
```

For every unread message, we provide a link and we tell angular that when user clicks it (`ng-click` attribute) the
function `markRead()` has to be called with the parameter `$index`. We define that function inside the angular
controller and attach it to the `$scope`:

```js
    // ...

    // mark messages read
    $scope.markRead = function (index) {
        var id = $scope.messages[index].id;
        $http({
            method: 'POST',
            url: '//127.0.0.1:8000/api/inbox/'+id+'/read/',
            xsrfHeaderName: 'X-CSRFToken',
            xsrfCookieName: 'csrftoken'
        })
        .success(function (data, status, headers, config) {
            $scope.messages.splice(index, 1);
        })
        .error(function (data, status, headers, config) {
            // something went wrong :(
        })
    };
    // ...
```

The parameter passed to `$http` contains all the logic needed to retrieve the csrf token from user's cookie and pass it
to Django inside the `X-CSRFToken` header. For my experience, I've never seen an easier way to do this (thank you so
much, Angular!). After retrieving the database id for the *index-th* message, we call the `/api/inbox/{lookup}/read/`
endpoint which marks that message as read. In case the request goes well (and this is where magics happen), we remove
the element from the `$scope.messages` - angular will remove that element from the DOM afterwards. No code. No explicit
DOM manipulation. Just fun.

Since Django Stored Messages api exposes an endpoint to mark all messages read, we provide a button to do exactly this.
The code for the button is very similar to the one to mark messages read:

```html
    <button type="button" class="btn btn-success" ng-click="markAllRead()">Mark all read</button>
```

This time the function name is `markAllRead` and we call it without parameters; the function is defined inside the
controller:

```js
    // ...

    // mark all read
    $scope.markAllRead = function () {
        $http({
            method: 'POST',
            url: '//127.0.0.1:8000/api/mark_all_read/',
            xsrfHeaderName: 'X-CSRFToken',
            xsrfCookieName: 'csrftoken'
        })
        .success(function(data, status, headers, config) {
            $scope.messages.splice(0, $scope.messages.length);
        })
        .error(function(data, status, headers, config){
            // something went wrong :(
        })
    };
    // ...
```

The csrf boilerplate is the same (for the record, this could be easily avoided using some advanced angular features)
and the logic is very similar: in case the request succeeded, the array of messages is cleared and the DOM reflects the
changes automagically.

References
----------

* [A working example is on GitHub](https://github.com/masci/notification_example)
* [Django Stored Messages documentation](http://django-stored-messages.rtfd.org/)
