---
title: {{ replace (substr .File.ContentBaseName 4) "_" " " | title }}
description: ""
category:
tags: []
date: {{ .Date }}
slug: {{ replace (substr .File.ContentBaseName 4) "_" "-" }}
rating: "★★★★☆"
cover: "/images/{{(substr .File.ContentBaseName 4)}}.png"
---

Intro...

<!--more-->

... more intro

## Notes
