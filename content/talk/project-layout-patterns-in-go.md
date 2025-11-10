+++
date = 2018-10-22T11:30:00Z
title = "Project Layout Patterns in Go"


[[conferences]]
name = "Golab"
date = 2018-10-22T00:00:00Z

[[material]]
name = "Slides"
url  = "https://speakerdeck.com/masci/project-layout-patterns-in-go-d18cf929-3642-4850-9499-1408c555e462"

[[material]]
name = "Video"
url  = "https://www.youtube.com/watch?v=3gQa1LWwuzk"

+++

You completed the tour, learned the language, got your pet project on Github.
Time to start a new project, a big one: how do you organize code, tests, docs?
During the talk I’ll try to save you some trial and error by sharing the lessons
learned during a year spent coding a non trivial Go project.

There’s no silver bullet to organize a non trivial Golang project, only good
practices and experience. You can try different things but when the project gets
traction, with a lot of people involved, it might be difficult to change
strategy. You may look at what others do in bigger, well known projects but size
matters and you can’t just do “what they do at Kubernetes”.
During the talk I’ll show some lessons learned while building from the ground up
a Golang project that now has more than 600 source files, dozens of packages and
the tooling needed to build binaries, packages and Docker images.

{{< youtube 3gQa1LWwuzk >}}
