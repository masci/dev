---
title: "How I manage a small team with the help of Trello"
date: 2018-02-03T16:47:03+01:00
draft: false
tags: [agile,management,trello]
url: "manage-with-trello"
---

This is not about agile management theory, something I know very little about; this is what I do to keep a small team of engineers on track without allocating 100% of my time to management. If you keep reading, you'll see how I didn't invent anything and how this is only "yet another way" of composing existing tools and practices in order to implement a process that works for me. So why another post?

Well the problem is, it costed to me a lot of time and trial-and-error to get here and I decided to share my experience so that others can take inspiration, specially if some of the following apply:

1. You're part of a small team, let's say 8 people or less.
2. Your team is responsible of delivering new projects and maintaining existing ones.
3. You have formal deadlines, objectives and goals.
4. You want some time to spend working on the product and not on management.
5. You want to keep things simple.

## Tools

My team is geographically distributed so we need remotely accessible tools to share informations. Beside the ubiquitous e-mail and instant messaging, the only management tool we use is Trello.

Being general purpose, Trello offers a tiny fraction of the features one can find in specialised tools like Jira and Asana but this is actually a key point if you want to keep the management overhead at minimum for your team.

## The basics

My approach is based on three building blocks:

1. The Backlog
2. The Sprint
3. The Roadmap

Let's see what those mean and how such blocks can be implemented with Trello.

#### The Backlog

{{< figure src="rmsuypnd3poxy7vzhky8.png" title="the Backlog in Trello" width="100%">}}

As a team, we don't usually work in isolation: interacting with Product Management, Customer Support, Documentation and other engineering teams is desirable and expected. The Backlog is the point of contact with the rest of the company and is the place where to put items like feature requests, bug reports or more in general any small, self contained task my team should work on - such items are represented with Trello cards. We sometimes put tasks ourselves on the Backlog, for example when priorities need to be discussed and evaluated.

A Trello board is a perfect fit to implement a Backlog, see [this basic example](https://trello.com/b/0A4qO3OV/backlog). The number and type of lists can vary a lot depending on team activities but at least two columns should be there:

1. _Triage_: is where new tasks are parked waiting for a priority to be assigned.
2. _Next_: is where tasks are sent after a priority is assigned, waiting to be assigned and scheduled during an upcoming sprint.

It's fundamental to allocate some time to review the Backlog board, how often depends on the workload. A fixed schedule works better for me but there are really no rules here. The review should involve managers and product managers who can help deciding priorities and engineers who can provide help on better defining the scope of a card, or the amount of work that a card implies.

#### The Sprint

{{< figure src="uzr6iot6nyegr8d1fvhe.png" title="the Sprint in Trello" width="100%">}}

The sprint board provides a picture of the current status of the team: what's done, who's working on what, what's waiting for a review and most important, what's blocked and why. For this to work, each card must have one or more owners, something we do on Trello by listing specific users in the "members" list. You can see how a Sprint board looks like in Trello [here](https://trello.com/b/zFlf1VII/sprint).

Most of the cards you see on the Sprint board come from the Backlog, specifically from the _Next_ column. This process is formalised with a planning meeting that usually happens right before a sprint starts but you can also let people pick tasks from there when needed, since cards are supposed to be already scheduled and have a priority assigned.

There's not much to discuss about the Sprint Board but there are a few caveats:

1. The _TODO_ column should contain only cards that are supposed to be done within the current sprint. Cards tend to pile up there, specially low priority ones so when this happens, I remove the owner and send them back to the Backlog, even on _Triage_: maybe the priority was wrong, the task poorly defined or even not needed in retrospective, that's ok.

2. All the cards must have an owner! It might seem obvious but since most of the times you look at the sprint board with some filter active, orphan cards can become invisible.

3. It's ok to skip the backlog board altogether and add a card directly on the sprint board: it's the case for bugfixes, or urgent tasks that pop up midpsprint but we try to keep this an exception.

#### The Roadmap

{{< figure src="38wlg3osr3dxpmbwip9z.png" title="the Roadmap in Trello" width="100%">}}

Backlog and sprint boards are great to track the work of a team because it's extremely easy to answer questions like "What are you working on right now?", "What's next?", "Is there any blocker?" but what if you want to know the overall status of a specific project? On Trello you can use labels to link a card to a project but cards might be spread across Backlog and Sprint and while you can still filter, you can't avoid to go back and forth the two boards. Roadmaps can add another dimension you can look at your team's cards from.

Each card in a roadmap board represents a macro feature, or an _Epic_, and must contain a brief description along with one or more checklists. Each item in a checklist is a link to a card that might be in the backlog or the sprint board, see [this Trello board](https://trello.com/b/WTSoEcD1/project-mayhem-roadmap) for an example.

You'd need one Trello board for each project you want to track; as for the backlog board, you can go wild with the number of lists but the following should always be there:

1. _Now_: epics that are currently under development.
2. _Next_: epics that should be done as soon as you have bandwidth.
3. _Later_: epics that have low priority and should be _eventually_ done.

Cards never leave a roadmap board, you're only supposed to move them across the lists, ideally from _Later_ to _Next_, then to _Now_ and finally archived (or moved to a _Done_ list).

Being a collection of tasks, epics represent a quite large chunk of work for a team but they might not be enough to track big features or long term goals; in this case you can group epics using a _Theme_ that in Trello words can be represented with a label. Examples of themes might be `version:1.0` or `feature:SAML`. You can then filter a roadmap board by label to see what's the status of the project in regard of one particular theme.

Managing a roadmap board requires some extra time, specially composing the checklists since it's all manual work, but in exchange you get a perfect view over the status of a project, something that might be extremely useful when discussing long term goals that span over multiple sprints or even over multiple quarters.

## Conclusions

I'm still evaluating the tradeoff between the extra work of maintaining roadmaps and the ability to answer questions about the big picture but overall it's working pretty good and proof is I could finally find the time to write this piece. Should you have any feedback, please reach out on Twitter @maxpippi.
