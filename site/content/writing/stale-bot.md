---
title: "Make a stale bot, learn GitHub Actions"
date: 2019-09-22T15:53:40+02:00
draft: false
---

A few months ago I had the chance to play with the first Beta of [GitHub Actions][6]
but for some reasons it didn't click for me: the point-and-click interface, the
HCL syntax and the confusion in the market place put me back to my comfort zone,
in the good company of the usual CI/CD systems we all have been using for years now.

Fast forward to a few weeks ago, when GitHub announced that Actions were finally
approaching general availability; a couple of days later, my personal GitHub
account was promptly enrolled in the Beta program. It was time to give it another
try, and this is the story of my second attempt at learning Actions.

## Introducing the Stale Bot

<img src="/images/dominik-scythe-Sot0f3hQQ4Y-unsplash.jpg" alt="A GitHub stale bot" title="Photo by Dominik Scythe on Unsplash" width="100%"/>

When I learn something new, it's vital for me to have some sort of prototype I
can change and evolve as I ramp up on the subject. That's why after few basic
experiments and a couple of easy projects ported from other CI/CD platforms, I
challenged myself to write a GitHub bot implemented in a single Workflow with
minimum recourse to 3rd party Actions.

The requirements are admittedly idiotic but they made a good excuse to learn
how to:

* use scheduled Workflows
* execute jobs conditionally
* interact with the GitHub API from a Workflow

## Requirements

A stale bot is an automated mechanism responsible for marking issues and pull
requests as "stale" when there's no activity around them after a given amount of
time. For the sake of simplicity, we'll limit the scope of the bot to issues
only.

The requirements are the following. For each issue on a GitHub repository:

1. Apply the label `stale` after 30 days without any kind of activity,
1. remove the label `stale` when there's activity on the issue,
1. close the issue if it's been `stale` for more than 15 days.

## Create the Workflow

First of all, let's create a GitHub Workflow. You can do it from the GitHub UI
or by simply creating a yaml file under `.github/workflows/` at the root
of your repo. You might have different Workflows defined for a single repo and
the name of the file can be whatever, in my case it is `stalebot.yaml`. We also
have to give our Workflow a name that will be displayed in GitHub UI: let's call
it "Stale Bot". Our yaml file will look like this:

```yaml
name: Stale Bot
```

## Determine when the Workflow has to run

We have to tell GitHub when we want to run our Workflow and we can do it with the
keyword `on`. According to our requirements, the Workflow should be triggered by
different events:

* Every night, to attach the `stale` label and to close issues that've
been stale long enough.
* Every time the issue is edited, or a label or milestone is attached.
* Every time somebody leaves a comment on the issue.

To run a Workflow periodically at a given time, we can use the `schedule` keyword,
followed by one or more intervals defined in [POSIX syntax][0], like in a `crontab`
file. For example, if we want our bot to run every night at midnight, we add:

```yaml
on:
  schedule:
    - cron:  '0 0 * * *'
```

To also run the workflow every time an issue has activity, we add the `issues`
keyword, so that the previous snippet will look like this:

```yaml
on:
  schedule:
    - cron:  '0 0 * * *'
  issues:
```

GitHub API sends different events for different things that happen to issues:
by leaving the `issues` mapping empty, we accept the defaults. You can
see what's the default for any value of the `on` keyword in the docs
[page about events][1]; in our case, the default value for `issues` is
["any possible kind of interaction"][2] which is a bit too much.
That's not a problem because we can limit the kind of events that trigger our bot
by listing them explicitly in a sequence under the keyword `types`, like this:

```yaml
on:
  schedule:
    - cron:  '0 0 * * *'
  issues:
    types: [edited, milestoned, labeled]
```

Our bot will be then summoned whan an issue is edited, a milestone is added or a
label is attached. Since we also want the bot to be activated when somebody
leaves a comment on the issue, we need one more event on our `on` mapping,
specifically the `issue_added` event:

```yaml
on:
  schedule:
    - cron:  '0 0 * * *'
  issues:
    types: [edited, milestoned, labeled]
  issue_comment:
```

The default `types` for the `issue_comment` event work for us, no need to add a
`types` keyword here.

## Define the jobs

A Workflow consists of a set of **jobs** that will be run in parallel; each
job consists of a sequence of **steps** that will be run one after another.
We're going to define a single job named `stale-bot-logic` containing
one step for each requirement we have, for a total of three steps. Since we can
choose the platform on which our workflow will run, we pick Linux, specifically
`ubuntu-latest`. Let's add this to our workflow yaml definition:

```yaml
on:
  schedule:
    - cron:  '0 0 * * *'
  issues:
    types: [edited, milestoned, labeled]
  issue_comment:

jobs:
  stale-bot-logic:
    runs-on: ubuntu-latest
    steps:
```

A step can either run a command on the host running the Workflow, or run a
Docker container or execute a special kind of node.js applications called _Actions_.
There are a lot of Actions available, ready to be included in a Workflow;
to have a better idea you can browse the GitHub [market place][3] - chances are
that somebody else already solved a problem you're having and no wheels will be
reinvented that day.

### Mark issue stale

The first step of the job will be marking an issue as stale after a certain
amount of time: this means we need to query the GitHub API and guess
what: there's an Action for that. The Action we need is called
`actions/github-script` and it'll be the only one we're going to use. It exposes
a Javascript client that can make API calls straight from the yaml code. The
Workflow will then look more or less like this (`on` section omitted for brevity):

```yaml
jobs:
  stale-bot:
    runs-on: ubuntu-latest
    steps:
      - name: Mark stale
        uses: actions/github-script@0.2.0
        with:
          github-token: ${{ github.token }}
          script: // Here it goes some Javascript code that uses the GitHub API client
```

The `uses` keyword will tell the CI system we want to use a certain action at the
version `0.2.0`. Actions may expose few parameters you can use to configure their
behaviour (refer to each Action's docs for the details) and that's what the `with`
keyword is for: we'll pass in a valid GitHub token using the param `github-token`
and some Javascript code using the param `script`.

The logic for this job will be the following:

1. Get a list of all open issues.
1. For each one, get the time and date of the last update.
1. If the difference between now and the last update for an issue is above a certain
threshold, attach a label called `stale` to it.

The code will look like this (I'll paste only the job definition for brevity):

```yaml
- name: Mark stale
  uses: actions/github-script@0.2.0
  with:
    github-token: ${{github.token}}
    script: |
      // Fetch the list of all open issues
      const opts = github.issues.listForRepo.endpoint.merge({
        ...context.repo,
        state: 'open',
      });
      const issues = await github.paginate(opts);

      // How many days of inactivity before we mark the issue stale
      const elapsedDays = 30
      // Get the time window in milliseconds
      const elapsed = elapsedDays * 24 * 60 * 60 * 1000;

      const now = new Date().getTime();
      for (const issue of issues) {
        // Ignore issues below the threshold
        if (now - new Date(issue.updated_at).getTime() < elapsed) {
          continue;
        }

        // If we got here, mark as stale.
        github.issues.addLabels({
          ...context.repo,
          issue_number: issue.number,
          labels: ['stale']
        });
      }
```

That's pretty much all the code we need but there's one detail we need to take
care of: according to the `on` configuration we defined earlier, the Workflow
will run every day as a cron job and also every time an issue is updated. There's
no need to have the "Mark Stale" job run every time an issue is updated (obviously
an issue that gets updated is not stale) and we want to skip its execution in this
case. In other words, we want to run the step only when the Workflow was kicked
off by the cron scheduler. To do so, we can use the `if` keyword on our job:

```yaml
- name: Mark stale
  uses: actions/github-script@0.2.0
  if: github.event_name == 'schedule'
  with:
    ...
```

The job will be then skipped unless the statement in the `if` value is true.
That's Javascript code and you can access several objects called _contexts_ from
there, in this case we're accessing the `event_name` field of the
[`github` context][4].

### Remove stale label

Sometimes a stale issue gets noticed and some activity kicks off again. In this
case, we need to remove the `stale` label as soon as possible. We use
the `github-script` Action again and this time we'll ask the CI system to
skip the job unless the source event is either an issue update or a new comment.
Note how we use a slightly more complex `if` clause here:

```yaml
- name: Remove stale
  if: github.event_name == 'issues' || github.event_name == 'issue_comment'
  uses: actions/github-script@0.2.0
```

This time the logic will be the following:

1. Get all the labels of the issue that triggered the Workflow.
1. Search whether it has a `stale` label and remove it.

The final job definition will than be:

```yaml
- name: Remove stale
  if: github.event_name == 'issues' || github.event_name == 'issue_comment'
  uses: actions/github-script@0.2.0
  with:
    github-token: ${{github.token}}
    script: |
      // Fetch the list of labels attached to the issue that
      // triggered the workflow
      const opts = github.issues.listLabelsOnIssue.endpoint.merge({
        ...context.repo,
        issue_number: context.issue.number
      });
      const labels = await github.paginate(opts);

      for (const label of labels) {
        // If the issue has a label named 'stale', remove it
        if (label.name === 'stale') {
          await github.issues.removeLabel({
            ...context.repo,
            issue_number: context.issue.number,
            name: 'stale'
          })
          return;
        }
      }
```

### Close stale

We've one last job left to complete our stale bot: an issue should be closed in case
it's been marked as stale for a certain amount of time. The logic is the following:

1. Load all the issues having the `stale` label attached.
1. For each one, compute the difference between now and the last update of the
issue (we assume it concides with the moment the `stale` label was added)
1. If the difference is above our threshold, close the issue.

The job definition looks like this:

```yaml
- name: Close stale
  if: github.event_name == 'schedule'
  uses: actions/github-script@0.2.0
  with:
    github-token: ${{github.token}}
    script: |
      // Fetch the list of all open issues that have the 'stale' label
      // attached
      const opts = github.issues.listForRepo.endpoint.merge({
        ...context.repo,
        state: 'open',
        labels: ['stale'],
      });
      const issues = await github.paginate(opts);

      // How many days of inactivity before we close a stale issue
      const elapsedDays = 15
      const elapsed = elapsedDays * 24 * 60 * 60 * 1000;

      const now = new Date().getTime();
      for (const issue of issues) {
        // Ignore issues below the threshold
        if (now - new Date(issue.updated_at).getTime() < elapsed) {
          // This is how to print debug informations
          console.log('skip issue ' + issue.number);
          continue;
        }

        // If we arrive here, the issue has to be closed
        console.log('closing issue ' + issue.number);
        await github.issues.update({
          ...context.repo,
          issue_number: issue.number,
          state: 'closed'
        });
      }
```

You can find the complete workflow [here][5].

## Caveat and final considerations

<img src="/images/lenin-estrada-OI1ToozsKBw-unsplash.jpg" alt="A happier GitHub bot" title="Photo by Lenin Estrada on Unsplash" width="100%"/>

First thing first, this bot shouldn't be considered feature complete by any mean,
the goal was just ramping up up on a new technology without getting bored. Now that
you're more familiar with GitHub Actions, a bunch of different and possibly more
efficient workflows to solve the same problems might have popped into your mind
already; if you need a stale bot for your project, I wouldn't see any problem
on keep going this way and make the implementation better.

That said, there's a subtle problem with this approach that I think it's worth
discussing: the code we wrote is just a piece of text defined in a yaml file that
GitHub will happily pass to the entrypoint of the `actions/github-script` action.
This means few terrible things:

* No syntax highlighting
* No code validation
* No tests

Which can turn into respectively:

* A miserable user experience for authors and contributors
* A frustrating development cycle
* Bugs and poor performance

This is actually very interesting because this problem can help us to better
define what the boundaries of a Workflow should be. I'd phrase like this:

> If a workflow contains more than two lines of logic in plain text form, it's
time to write a custom Action.

With a custom Action (whether Javascript or container based) you can
dramatically simplify the workflow definition while having tests and the
ability to exercise your code locally. You could also look at this problem
symmetrically:

> If your problem can be solved with two lines of logic, writing a custom Action
is probably overkill

I'm a big fun of custom Actions and I enjoy Typescript, but I have to say they
don't come for free: you basically need to write a full fledged
Node application and whether this is a problem or not, it's up to you.

[0]: https://pubs.opengroup.org/onlinepubs/9699919799/utilities/crontab.html#tag_20_25_07
[1]: https://help.github.com/en/articles/events-that-trigger-workflows#about-workflow-events
[2]: https://help.github.com/en/articles/events-that-trigger-workflows#issues-event-issues
[3]: https://github.com/marketplace
[4]: https://help.github.com/en/articles/contexts-and-expression-syntax-for-github-actions#github-context
[5]: https://github.com/masci/stalebot/blob/master/.github/workflows/stalebot.yml
[6]: https://help.github.com/en/articles/about-github-actions
