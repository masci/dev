+++
date = 2017-01-24T11:30:00Z
title = "How to port your Python software to Go without people noticing - a real story"
lang = "en"

[[conferences]]
name = "Golab"
date = 2017-01-20T00:00:00Z

[[material]]
name = "Slides"
url  = "https://speakerdeck.com/masci/how-to-port-your-python-software-to-go-without-people-noticing"

+++

Success stories about rewriting Python applications in Go are not big news anymore. 
The pros and cons are well known, best practices are in place, the standard library 
is there to help. But what if there’s some Python code you would like to keep or worse, 
some you can’t get rid of? 

When we chose to port the Datadog Agent to Go, we had a 
requirement to maintain our plugin system in Python. During the talk we will share 
stories about cgo, the GIL and the quest for performance as we bridge multiple languages 
in a single application.
