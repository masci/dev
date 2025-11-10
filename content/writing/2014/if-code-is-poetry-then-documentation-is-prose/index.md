---
title: "If code is poetry, then documentation is prose"
description: ""
category:
tags: ["python", "docs"]
date: "2014-05-24"
url: "if-code-is-poetry-then-documentation-is-prose"
---

## What is poetry?
Before starting, let’s agree on some terminology. Obviously poetry is a literary form, a way to write down things I would say,
but how do you recognize it, how can you say that is poetry and this is not? Well, according to my literary heritage (I’m italian)
something is poetry when it has verse, rhyme and stanza; let’s see what they are and how we can recognize them.

### Verse

A verse is a sequence of words on a single line; those sequences must follow strict rules that determine their length and
contents in terms of syllables. The verse gives rhythm to the composition.

### Rhyme
A Rhyme is the equality of the sound of different words and it gives melody to the composition.

### Stanza
A group of verses; even these groups follows strict rules about the number of verses they might contain and the way they are connected
together, with or without rhymes.

Let’s see an example that follows all these poetry rules, a stanza from *Il gelsomino notturno* (*Night Blooming Jasmine*) by Giovanni Pascoli:

    E’ l’alba: si chiudono i petali
    un poco gualciti: si cova,
        dentro l’urna molle e segreta,
        non so che felicità nuova.

and its english translation:

    It’s dawn: the petals, slightly worn,
    close up again—each bud to brood,
        in its soft, secret urn,
        on some yet-nameless good.

Why do we like it? It has rhythm, sounds good spoken aloud, contains beautiful words, has a strange but pretty aesthetic form once written
down (the tabs are part of the verses) and has a powerful sexual metaphor hidden in it - you get it even if it’s not explicit. You know
it’s there and in fact this is what the author really meant to tell you. Would it be the same if the author pulled in some detail about the
fact? Hell, no!

Now what about something like this:

```py
    lock = threading.lock()
    lock.acquire()
    try:
        print("Let's get critical")
    finally:
        lock.release()
```

Do you get the aesthetic, the rhythm, the meaning? Maybe the meaning, but the rest is rather ordinary. And what about this?

```py
    lock = threading.lock()
    with lock:
        print("Let's get critical")
```

Do you spot the difference? If not, you’re taste is broken, otherwise you get the point and we can agree that code could be a
form of poetry, or at least that it's possible to write code in a poetic fashion.

# What’s wrong with poetry?
Poetry is one of the highest but most challenging forms of art: to cut a long story short, poetry is difficult for the most
of us. Take this for example:

    Da un immoto fragor di carrïaggi
    ferrei, moventi verso l’infinito
    tra schiocchi acuti e fremiti selvaggi…
    un silenzio improvviso. Ero guarito.

its english translation:

    Out of a motionless infernal
    shudder and clang of steel on steel
    as wagons moved toward the eternal,
    a sudden silence: I was healed.

> Wat?

As poets do, most developers can write beautiful and elegant code, they really enjoy doing it but how hard is reading their works?
Let's see an example of well written code:

```py
    from itertools import chain, islice

    def chunks(iterable, size, format=iter):
        it = iter(iterable)
        while True:
            yield format(chain((it.next(),), islice(it, size - 1)))
```

> Ok, give me a sec...

To grasp the meaning of a code snippet you have to read it carefully, examining the overall context, reading the code
that comes before and after, trying to guess programmer’s mind at last. Very few people will be motivated enough to go
through the process and the result will vary between unhappy users, angry users and former users. Don’t let the poetry
alone!

# Here comes the prose
Prose is *the other* literary form, sometimes defined as “everything is not poetry”.
[Eugenio Montale](http://en.wikipedia.org/wiki/Eugenio_Montale) was one of the greatest italian poets and he won the
Nobel Prize in Literature in 1975; during an interview he was once asked about the differences between prose and poetry
and he stated that while Prose is *horizontal*, Poetry is *vertical*. Poetry is short, emotional, made of evocative
and powerful words, it can bring you different feelings so quickly you can barely notice. Prose is long, tells you
a story with clear and useful words, takes your hand and walks along with you towards the meaning.

You get the point, prose can explain poetry just like documentation can explain code.

# Why programmers don’t write prose
I have an educated guess: writing docs is hard and life consuming. As a core developer of a mid-sized open source
project I found myself writing way more lines of docs than code. I used to be a poet but I had to turn into a writer
for the occasion. But it’s not only a matter of quantity, writing docs is also challenging for a series of issues:
you may not be a native english speaker and writing good docs is hard in that case; or you may be a native english
speaker but you have to wisely choose your words to make your docs understandable for non native speakers; you have
to write examples, tutorials and advertise them; you have to make screenshots; you have to setup different environments;
you have to put yourself in users’ shoes; you have to argument with strangers on the internet.

# Why you should write prose
First of all, prose and poetry have different audience. Code Poets will likely dig into your code whether or not the
project has documentation; humans just start from the documentation. More humans, more users; more users, more
contributors; more contributors, less work for everyone :)

# How to write prose
Writing docs when most of the code is already in place and project features almost complete is difficult and disheartening:
you have to invent good stories for your users as an aftermath, you have to review the code and fill the gaps in your
docstrings, you have to put in place examples and use cases. What you should do instead is writing documentation along
with or better before the code: someone refers to this as *Documentation Driven Development*. Writing docs before the code
can give you precious hints about the idea you are going to implement: as a rule of thumb, if you struggle to describe your
idea into the documentation, maybe that is not a good idea. Write the stories, write the docstrings then write the code
(insert tests in the process where you’re more comfortable with).

# Lessons learnt as a prose writer

* Always write docstrings for your and your team’s sanity and write them early, even before the code.
* Write good docs and make them available to the internet.
* Write tutorials, a lot of tutorials.
* Write articles, show people how to use your code.
* Listen to your users, there are no stupid questions, only stupid developers not answering.
