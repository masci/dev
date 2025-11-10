---
title: "Stop asking users for passwords and let OAuth2 do the job instead"
description: ""
category:
tags: ["django", "oauth2", "security", "authentication"]
date: "2013-11-10"
url: "stop-asking-users-for-passwords-and-let-oauth2-do"
---

## Who said authentication?

If you have an HTTP endpoint which requires authentication, chances are that you're using HTTP Basic auth or
Digest. This was the case, you should immediately stop asking users for their passwords - let's see why.

It's not a matter of security per-se, I mean HTTP Basic auth is reasonably fine in many situations where the
transmission occurs over SSL and it has some pros: it's easy to implement and it uses the well known
[HTTP Authorization header](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.8), largely supported
by almost all the HTTP clients out in the wild.

But what if an user falls in a SSL based man-in-the-middle attack (see section 4.8 of the
[rfc2617](http://www.ietf.org/rfc/rfc2617.txt))? Her credentials would be compromised and
when you realize it, you could be forced to suspend user's account or reset her password at minimum.

And what if your web resource is accessed by third party services? You would force users to reveal their password
to another party to let them access their own data, and if your users are smart, they would feel very uncomfortable
with that. Moreover, if some user changes her password on your server, access would be denied to other services.

You can avoid the hassle using a token instead of an username and password pair: you can change it whenever you
want without users caring about, you can revoke it at any time in case of trouble without suspending user accounts,
you can give it to other parties with the consensus of your users without making them reveal any secret. Token
authentication is practical and secure, but how to implement it?

Well, I have some good news: there's a protocol for that, and that protocol has an RFC standard. It's called
OAuth2.

## OAuth2 FTW

It was not so much time ago that I've been considering OAuth2 solely a tricky protocol used to authorize a
service to access data provided by another service, and there are still people with the same misbelief. OAuth2
can easily replace Basic auth over SSL and there are plenty of good libraries that make the implementation a charm.

The OAuth2 world is token-centric: you have to retrieve an access token and use it on every request to the server
when authentication is required; for whatever reason you want, you can easily get another, brand new
token at any time (if the server supports it, this behaviour is optional indeed); you can give your token to
third parties and let them access your resources "as they were you"; every token may have a *scope*, a list of
things that it's allowed to do or do not; you can revoke the token at any time if you want (and so can the server).

OAuth2 offers different strategies (called *flows*) to let users retrieve their token, and actually some of them
are rather complex. Since we're trying to replace Basic auth, let's see an OAuth2 flow which is as much as
simple, the *Resource owner password-based grant*: users provide their username and password only once to obtain
an access token that can be used to access resources on the server.

You can easily try by yourself how this authentication method works using a real world OAuth2 provider as a server
and curl on your local machine as a client.

### Register an app

Go to [OAuth2 Playground](http://django-oauth-toolkit.herokuapp.com/), a fully functional OAuth2 provider written
in Django using [Django OAuth Toolkit](https://github.com/evonove/django-oauth-toolkit) library, follow the menu
*OAuth2 Provider --> Register an Application* and login with test/test username and
password. Give a name to your application (this is particulary useful in some flows), choose suitable values for
*client id* and *client secret* fields or leave the defaults (they're automatically generated in a safe way by
the system), choose *Public* as *Client type* and *Resource owner password-based* as *Authorization grant type*.
For the OAuth2 flow you're going to use there's no need of any *Redirect uris* so you can leave that field blank.
Hit *save* to finally register your application on the OAuth2 provider.

### Grab your token

Now we need a token to access resources on the Playground server. We will ask for a token for our Application
and the user *test* of whom we know username and password; the url where retrieve the token is */o/token*
(every OAuth2 provider has its own url mapping of course), let's go there with curl:

	curl -d "grant_type=password&client_id=your_client_id_here&username=test&password=test" http://django-oauth-toolkit.herokuapp.com/o/token/

If everything goes smooth, server will answer with something like this:

	{
	    "refresh_token": "a_refresh_token_here",
	    "token_type": "Bearer",
	    "scope": "example",
	    "access_token": "the_access_token",
	    "expires_in": 36000
	}

The server provided also a refresh token, a second token which can be swapped for a new access token
at any time before the current one expires. With the access token provided, we can authenticate against the server
using `Authorization` header and providing our access token. In this example, we're going to access detailed data
for our application through a proper endpoint:

	curl -H "Authorization: Bearer the_access_token_here" http://django-oauth-toolkit.herokuapp.com/api/v1/applications/your_client_id_here/

If the server successfully authenticate and the scopes provided are enough to access the data we requested, we will
receive a response in json format similar to this:

    [{
        "model": "example.myapplication",
        "pk": 2,
        "fields": {
            "client_type": "public",
            "user": 2,
            "client_id": "",
            "name": "My own APP",
            "authorization_grant_type": "password",
            "client_secret": "",
            "description": "",
            "redirect_uris": ""
        }
    }]

From now on, our credentials are represented by the access token, with benefits for both users and the service provider:
the token can be revoked by the server if needed (account suspension, abuse of the service, etc.), or can be revoked
by ourselves if we do not want to access that service anymore. Or it can be refreshed with a new token if we fear our
old one would be compromised. We had to provide username and password just once but now we can use the token anywhere
we need to authenticate: a third party service or a mobile app for example.

## So what?

Even if security level is similar, using OAuth2 allows much more control and flexibility than using username and password,
provided that communication occurs over SSL. Well, the server has to manage a slightly more complex workflow when using
OAuth2 (think about applications registration and tokens management), but since OAuth2 is a standard described in
[RFC6749](http://tools.ietf.org/html/rfc6749) there are plenty of libraries and software components which implement the
protocol and can be used to drastically reduce the code needed.
