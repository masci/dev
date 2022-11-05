---
title: "Rapid Prototyping Web Services in Python: make it, don't fake it!"
date: 2022-11-05T10:22:34+01:00
tags: ["python"]
images:
  - "/images/undraw_fast_loading_re_8oi3.png"
---

{{<figure src="/images/mark-konig-1UMrSoItdDE-unsplash.jpg" title="Photo by Mark König on Unsplash" width="100%">}}

From Wikipedia:

> Rapid prototyping is a group of techniques used to quickly fabricate a scale model of a physical part


## Good enough

Trying out a physical product for a fraction of its cost can be extremely effective, in fact we do
the same with software. We draw **sketches** and **wireframes** for websites or applications,
and sometimes they're clickable; you can see legit data and images, the whole user experience closely
resembles the "real thing". But it's not as common to interact with a real web service at an early
stage of the prototyping process, let's see why.

Recently I was validating an idea for a security software with some friends, and after hours and
hours of user stories, countless diagrams and workflows, it was time to prototype the mobile
application. We decided to invest very little time on it: in the end, an extremely rough UI would
have sufficed, and the backend could be mocked within the app itself, right? Right?

Well, that would be the norm. A web service has a basic set of requirements that puts a lot of
overhead onto the prototyping process: it needs to be available on the internet, which implies you
need an IP address, a domain name, a web server running 24/7 and possibly a way to store data... All
of this to mock one component of the final product. No wonders that the common way to prototype a web
service is pretending there is one, with the client making calls to itself and self-serving fake
data. But I think there's a better alternative, and it's almost as fast.

## Fast enough

Curious to see if I could come with a better solution than hardcoding a mocked service into our
client application in the same amount of time, I started coding a full-fledged backend application to
serve our API. I picked the fastest tools I know for this job: [Poetry](https://python-poetry.org) to
manage the Python project, [faker](https://faker.readthedocs.io/en/master/) to generate dummy data,
[FastAPI](https://fastapi.tiangolo.com) to implement the endpoints,
[uvicorn](https://www.uvicorn.org) to serve the API and Docker to run the whole thing.

Starting from scratch, it took me a couple of hours to get the first endpoint return something from a
running container, probably slower than hardcoding the service in the client, but fast enough.
Problem was, this only worked on my laptop - far from ideal.

The ability to involve the stakeholders, your team and possibly early users and get feedback in fast,
short iterations is instrumental during this phase of the product development: we had an `apk` to
send around for testing, and attaching a long list of instructions about how to run it along with its
companion Docker backend container was not an option.

## Simple enough

I insisted, looking for a way to make the backend container available on the internet and allow
testers to validate the product by setting up the mobile app alone. This was quite the opposite of
"rapid prototyping", but I wasn't blocking the rest of the team and being in between switching jobs I
had some time to invest - and the investment eventually paid off.

Do you remember the slogan _There's an app for that™_ (yes, it's a trademark)? Same goes for the long
list of turnkey solutions offered by the many cloud providers around: do you need to expose a running
container to the Internet? There's a service for that!

I went with Google Cloud Run because I'm most proficient in GCP technologies, but you can easily find
its AWS and Azure cousins. The mechanism is the same for all those services:
- Put your web service in a container
- Push the image to the registry they tell you
- The cloud provider creates a third-level domain name for your service, pulls the image, runs the container, end of the story.

That's exactly that simple! It's so simple that (like the big git-ops nerd I am) I automated
everything using Github actions. Push a commit to `main`? The action will build the container, push
to the registry, tell GCP to update the Cloud Run instance. Push to `dev`? Same story, but deployment
goes in a different, "staging" Cloud Run instance (ok I didn't need this one, that was purely for
fun).

## Wrapping up

I published my project on Github as a
[repository template](https://github.com/masci/fastapi-template): the README and the code should
provide the technical context I didn't want to add here. If you need to quickly deploy a web service,
give it a shot!
