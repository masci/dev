+++
date = 2017-12-30T23:33:29+01:00
title = "A Python and a Gopher Walk into a Bar - Embedding Python in Go"
lang = "en"

[[conferences]]
name = "GothamGo"
date = 2017-10-05T10:00:00Z

[[conferences]]
name = "PyGotham"
date = 2017-10-06T10:00:00Z

[[material]]
name = "Slides"
url  = "https://speakerdeck.com/masci/a-python-and-a-gopher-walk-into-a-bar-embedding-python-in-go-1"

[[material]]
name = "Video"
url  = "https://www.youtube.com/watch?v=egSvw7xYw9s"


+++

Success stories about rewriting Python applications in Go are not big
news anymore. The pros and cons are well known, best practices are in
place, and the standard library is there to help. But what if you want
to keep some of your Python code? When we chose to port the Datadog
Agent to Go, we needed to maintain support for our existing library of
plugins written in Python. During the talk we will share lessons
learned from our experiences with cgo, the GIL and the quest for
performance as we bridge multiple languages in a single application.

{{% youtube egSvw7xYw9s %}}
