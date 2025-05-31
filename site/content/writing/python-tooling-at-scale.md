---
title: "Python Tooling at Scale: LlamaIndex’s Monorepo Overhaul"
date: 2025-05-21
tags: ["python", "github"]
---

**This blog post was hosted in the LlamaIndex Blog, you can
[read it here](https://www.llamaindex.ai/blog/python-tooling-at-scale-llamaindex-s-monorepo-overhaul).**


When we talk about LlamaIndex, we’re actually referring to an ecosystem consisting of more than 650 Python packages,
mostly Integrations and Packs. All these packages share a single GitHub repository, what engineers fondly call a
“monorepo”. In this article, we’re going to introduce LlamaDev, our new tool for managing monorepos at scale, and
explain the challenges we ran into with existing tooling to get us to this point.

## The challenge: 650+ dependency trees

Each Python package in the monorepo is published on PyPI and comes with its own pyproject.toml file. For those not
familiar with the ecosystem, a pyproject.toml file for a modern Python package is a one stop shop defining several
aspects of the package lifecycle: its dependencies, its release version number, the Python and operating system
versions supported, and how tools like linters and type checkers should behave with that specific package. Most
packages have tests, and some have an additional list of development dependencies.

Integrations and Packs have some code in common, shipped through a foundational package called llama-index-core that
pretty much every package in the ecosystem depends on. Additionally, they can depend on each other, for example
llama-index-llms-azure-openai requires llama-index-llms-openai to work. As you can imagine, this has a few implications
for the testing strategy, such as:

  - If we touch llama-index-core in a pull request, tests for most of the packages in the monorepo need to be triggered
  to ensure they keep working after that change
  - If we touch something like llama-index-llms-openai, we need to find which are the packages depending on it and test
  them to ensure they still work

## Developing is easy(ish)

The experience of developing in one of these packages varies. Sometimes we work on a single, self-contained Integration
with no additional LlamaIndex dependencies other than llama-index-core. That’s the happy path, where we test our code
against a certain version of the core package and if it works, it works.

Sometimes we work on a package that has dependents in the monorepo. In this case, we need to ensure our changes work
for the package itself without disrupting other packages. Sometimes we work on the core package, where we need to be
extra careful not to introduce breaking changes that might impact the whole monorepo.

Last but not least, we do our best to support multiple Python and operating system versions in our packages, which for
the vast majority means Python 3.9 all the way up to 3.12+ on three operating systems: Windows, Mac and Linux.

## Maintaining is hard

The first thing you want in a monorepo is consistency. Even a slight difference in the way tests are executed or
packages are built can complicate cross-platform support and turn batch operations across the monorepo into a nightmare.
The risk of ending up with 650+ slightly different packages is very high, specially in the Python ecosystem, so the
first tool we brought in was a Python project manager.

A Python project manager usually relies on the pyproject.toml file to provide features like building and publishing a
package, managing dependencies and lockfiles, defining scripts and automation tasks, preparing virtual environments to
work on the code in isolation with different version of Python.

## First iteration: Poetry for individual packages

Our tool of choice was Poetry: feature complete, very well established in the Python community, it served us well for
over than a year. Working with Poetry on a single package is a breeze: with its expressive command line interface it
can set up specific Python versions and build virtual environments, so that running the tests while iterating on the
code becomes very easy. But don’t get too excited: we work on a monorepo, where even the easiest things become complex.

While Poetry is great to work in isolation on a single package, it doesn’t really know about package dependencies. For
example, it cannot know that a certain package depending on us actually sits in a sibling directory of the same git
repository. Being in the same repository, we could easily trigger the tests to verify that our changes upstream didn’t
break the dependents, but Poetry just doesn’t have the information to do this.

## First iteration: Pants for build management

And thus we had to introduce another tool: we needed a build system, preferably designed to explicitly work within a
monorepo. A build system is a piece of software that automates the process of building source code into artifacts,
taking care of all the intermediate steps. For a Python codebase, this means to set up a virtual environment for a
certain Python version, fetch and install dependencies, run the tests, build the wheel packages, and optionally to
publish them to some global registry like PyPI.

We chose Pants for the job, and again it served us well for more than one year. Pants explicitly supports Python
projects and is particularly smart in detecting which tests to trigger across multiple packages in the monorepo when
the upstream has changed. It also comes with a powerful caching system that can sensibly speed up the setup of each
build environment.

## Version 1.0 worked for a year

To recap, for well over a year this was the tech stack used in the LlamaIndex monorepo:

  - Poetry to manage the single Python projects.
  - Pants to orchestrate testing across different packages (we didn’t use the build features).
  - Github actions to automate Pants and Poetry runs on pull requests and git pushes.

Our setup was working, but admittedly had a few minor flaws. The problem with minor flaws is that they tend to become
less and less minor as the system scales. A one-second operation on a single package doesn’t seem much, until you have
to repeat it 650 times and now an automated job takes 10 minutes.

## Problems of scale: build speeds and caching servers

Let’s start with Poetry. The tool works pretty well and we could express sophisticated dependency requirements, but the
dependency solver can be very slow at times. Add it to the fact that pip is the only installer available, and the
overhead working on a package can be measured in seconds. Again, you won’t probably notice working on a single package,
but without Pants caching virtual environments, an extensive run of unit tests on the monorepo could take a very long
time.

Speaking of Pants, it came with its own set of flaws. The first one we encountered was setup: while defining targets
and dependencies is a hell of a task, but at least something you do once then profit, the caching system was more
demanding. We had to host a service on AWS behind a public address so it could be used from Github workflows. And the
caching server is not something you deploy once and forget. We had to scale it up a couple of times, add two redundant
instances behind a load balancer, resize storage a couple of times. Sometimes Github workflows failed with weird errors,
only to find out later that the caching server was unresponsive or behaving erratically. All understandable, but given
the size of our team, it was expensive to maintain in terms of resources and money.

## Problems of control: logging, inconsistency and more

And that’s not all: Pants’ sophisticated caching comes with an additional price. The Python virtual environments used
during a build are managed by Pants internally with pip. You don’t have much control over it, the environments are not
easily accessible, and this has a few implications.

  - First of all, Pants understands and can rely on Poetry configurations, but it doesn’t invoke it directly.
  This means a test run in the CI is different from a test run invoked locally with Poetry.
  - Second, Pants logging is really verbose, and messages related to installation or dependency problems are not easy to
  find and debug. Depending on the type of the error, at times even tests failures can be hard to find in the logs.
  - Lastly, running Pants locally is not easy and we don’t ask contributors to do that, but this means sometimes pull
  requests show failures that cannot be reproduced locally, making the developer experience quite poor.

## Why we needed version 2.0

To recap, these are the problems we were facing as the size of the monorepo increased:

  - Slow iterations due to dependency management struggles.
  - Maintaining the build system was a burden.
  - Debugging CI runs was hard.
  - Contributing to the project was hard.

Looking back at our past Slack conversations, it’s been a while since we started fiddling with the idea of replacing
pip with uv to take advantage of its huge speedups at install time, but we only recently had time available to make a
serious attempt. Here’s what happened.

## Migrating projects to uv from Poetry

The original task was quite vague, it essentially read something like “use uv everywhere so we can to shorten the CI
feedback loop”. We didn’t want to get rid of existing tooling, just introduce a new one that looked promising.

Pants has a section in the docs mentioning uv, so we were optimistic there could be a way to make the two coexist. That
optimism lasted about a hour, the time needed to go through the docs, browse some threads in the Pants Discord server
and realize that no, Pants cannot use uv instead of pip internally. There’s a third-party plugin that does that, but we
didn’t want to add another layer of complexity on something that we already barely controlled, so we canned that idea.

Plan B: we could still call uv within the Poetry projects, so at least we would speed up local development, no? No.
Poetry doesn’t support uv and it won’t in the foreseeable future. Luckily uv is much more than a replacement for pip,
it’s actually a very powerful Python project manager. So we took it for a spin and migrated a few packages to see what
changes would be required. This got off to a very good start. If you have uv installed, migrating a package is
literally one command you run on the folder containing the pyproject.toml file:

uvx migrate-to-uv

We won’t say we were impressed by how fast it was to install dependencies for a package, because that was expected.
What we fell in love with was everything else: uv is simple, tidy, expressive. uv run -- is so powerful you won’t need
to activate a virtual environment in your life ever again. Some of us, Homebrew hardcore users, let a Python project
manager tool install a Python distribution for the first time ever. It didn’t take us very long to make the call and
say goodbye to Poetry. If the developer experience was so good for us, contributors would certainly appreciate.

## How to manage builds in 2.0?

Still we weren’t happy. The CI still wouldn't benefit from uv and contributors would still have to struggle with cryptic
failures in pull request checks. Something was itching, and you don’t work on something like LlamaIndex if you don’t
scratch. So we scratched.

We are not so crazy to think we could rewrite something like Pants and replace it, but we were only using about 20% of
Pants features, and we definitely are more than 20% crazy. So we rewrote the code detecting the dependency graphs of all
the packages in the monorepo, relying on the pyproject.toml files alone. On top of that, we added some code to detect
what packages should be tested in every pull request, given the files that were changed. Then some code to invoke pytest
and grab only the logs that matter. Then some logic to avoid failing the CI jobs with untestable packages or unrelated
failures. Then we decided all this code could be turned into a custom made tool.

## Introducing LlamaDev

We’ve called our new tool LlamaDev and any contributor can run it from a clone of the monorepo with uv run llama-dev.

The list of improvements we introduced since we switched from Poetry+Pants to uv+LlamaDev is long:

  - Average time for a complete test run (without computing coverage) of all 600+ packages is about 20% faster,
  corresponding to about 6 minutes less.
  - Depending on the specific package, partial test runs are even faster. For example, changing an integration with
  many dependants like llama-index-llms-openai went from about 11 minutes down to 4.
  - Logs are much clearer now, and contributors can see exactly what happened if a check failed.
  - Contributors can run llama-dev locally and reproduce exactly what was done in the CI system if some debugging is
  needed.
  - Lockfiles are now checked into the repository for better build reproducibility.
  - We don’t rely anymore on external resources, all the computing needed is on Github actions.

And this obviously goes on top of everything uv does to improve the day-to-day when working on any Python project.
