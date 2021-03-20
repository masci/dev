---
title: "How to deal with API clients, the lazy way - from code generation to release management"
date: 2020-03-05T00:00:00+02:00
draft: false
tags: ["golang", "python", "openapi"]
---

**This blog post was hosted in the Arduino Engineering Blog, you can
[read it here](https://blog.arduino.cc/2020/03/05/how-to-deal-with-api-clients-the-lazy-way-from-code-generation-to-release-management/).**


The Arduino IoT Cloud platform aims to make it very simple for anyone to develop
and manage IoT applications and its REST API plays a key role in this search for
simplicity. The IoT Cloud API at its core consists of a set of endpoints exposed
by a backend service, but this alone is not enough to provide a full-fledge
product to your users. What you need on top of your API service are:

* Good documentation explaining how to use the service.
* A number of plug-n-play API clients that can be used to abstract the API from
  different programming languages.

Both those features are difficult to maintain because they get outdated pretty
easily as your API evolves but clients are particularly challenging: they’re
written in different programming languages and for each of those you should
provide idiomatic code that works and is distributed according to best practices
defined by each language’s ecosystem.
Depending on how many languages you want to support, your engineering team might
not have the resources needed to cover them all, and borrowing engineers from
other teams just to release a specific client doesn’t scale much.
Being in this exact situation, the IoT Cloud team at Arduino had no other choice
than streamlining the entire process and automate as much as we could. This
article describes how we provide documentation and clients for the IoT Cloud
API.

# Clients generation workflow

When the API changes, a number of steps must be taken in order to ship an
updated version of the clients, as it’s summarized in the following drawing.

{{< figure src="/images/workflow-gen.png" title="Generation workflow" width="100%">}}


As you can see, what happens after an engineer releases an updated version of
the API essentially boils down to the following macro steps:

* Fresh code is generated for each supported client.
* A new version of the client is released to the public.

## The generation process

### Part 1: API definition

Every endpoint provided by the IoT Cloud API is listed within a Yaml file in
OpenAPI v3 format, something like this (the full API spec is [here](https://github.com/arduino/clients-iot-api/tree/master/spec/v2)):

```yaml
 /v2/things/{id}/sketch:
    delete:
      operationId: things_v2#deleteSketch
      parameters:
      - description: The id of the thing
        in: path
        name: id
        required: true
        schema:
          type: string
      responses:
        "200":
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ArduinoThing'
          description: OK
        "401":
          description: Unauthorized
        "404":
          description: Not Found
```

The format is designed to be human-readable, which is great because we start
from a version automatically generated by our backend software that we manually
fine-tune to get better results from the generation process. At this stage, you
might need some help from the language experts in your team in order to perform
some trial and error and determine how good the generated code is. Once you’ve
found a configuration that works, operating the generator doesn’t require any
specific skill, the reason why we were able to automate it.

### Part 2: code generation

To generate the API clients in different programming languages we support, along
with API documentation we use a CLI tool called [openapi-generator]. The generator
parses the OpenAPI definition file and produces a number of source code modules
in a folder on the filesystem of your choice. If you have more than one client
to generate, you will notice very soon how cumbersome the process can get: you
might need to invoke openapi-generator multiple times, with different
parameters, targeting different places in the filesystem, maybe different git
repositories; when the generation step is done, you have to go through all the
generated code, add it to version control, maybe tag, push to a remote… You get
the gist.

To streamline the process described above we use another CLI tool, called
[Apigentools], which wraps the execution of openapi-generator according to a
configuration you can keep under version control. Once Apigentools is
configured, it takes zero knowledge of the toolchain to generate the clients -
literally anybody can do it, including an automated pipeline on a CI system.

### Part 3: automation

Whenever the API changes, the OpenAPI definition file hosted in a GitHub
[repository](https://github.com/arduino/clients-iot-api) is updated accordingly,
usually by one of the backend engineers of the team. A Pull Request is opened,
reviewed and finally merged on the master branch. When the team is ready to
generate a new version of the clients, we push a special git tag in semver
format and a GitHub workflow immediately starts running Apigentools, using a
configuration stored in the same repository. If you look at the main
configuration file, you might notice for each language we want to generate
clients for, there’s a parameter called ‘github_repo_name’: this is a killer
feature of Apigentools that let us push the automation process beyond the
original plan. Apigentools can output the generated code to a local git
repository, adding the changes in a new branch that’s automatically created and
pushed to a remote on GitHub.

## The release process

To ease the release process and to better organize the code, each API client has
its own repo: you’ll find Python code in
https://github.com/arduino/iot-client-py, Go code in
https://github.com/arduino/iot-client-go and so on and so forth. Once
Apigentools finishes its run, you end up with new branches containing the latest
updates pushed to each one of the clients’ repositories on GitHub. As the branch
is pushed, another GitHub workflow starts (see the one from the Python client as
an example) and opens a Pull Request, asking to merge the changes on the master
branch. The maintainers of each client receive a Slack notification and are
asked to review those Pull Requests - from now on, the process is mostly manual.

It doesn’t make much sense automate further, mainly for two reasons:

* Each client has its own release mechanism: Python has to be packaged in a
  Wheel and pushed to PyPI, Javascript has to be pushed to NPM, for Golang a tag
  is enough, docs have to be made publicly accessible.
* We want to be sure a human validates the code before it’s generally available
  through an official release.

# Conclusions

We’ve been generating API clients for the IoT Cloud API like this for a few
months, performing multiple releases for each supported programming language and
we now have a good idea of the pros and cons of this approach.

On the bright side:

* The process is straightforward, easy to read, easy to understand.
* The system requires very little knowledge to be operated.
* The time between a change in the OpenAPI spec and a client release is within
  minutes.
* We had an engineer working two weeks to set up the system and the feeling is
  that we’re close to paying off that investment if we didn’t already.

On the not-so-bright side:

* If operating the system is trivial, debugging the pipeline if something goes
  awry requires a high level of skill to deep dive into the tools described in
  this article.
* If you stumble upon a weird bug on openapi-generator and the bug doesn’t get
  attention, contributing patches upstream might be extremely difficult because
  the codebase is complex.

Overall we’re happy with the results and we’ll keep building up features on top
of the workflow described here. A big shout out to the folks behind
[openapi-generator] and [Apigentools]!

[openapi-generator]: https://github.com/OpenAPITools/openapi-generator
[Apigentools]: https://github.com/DataDog/apigentools