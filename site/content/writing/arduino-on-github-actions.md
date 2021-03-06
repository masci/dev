---
title: "Arduino on GitHub Actions"
date: 2019-11-14T15:53:40+02:00
draft: false
tags: ["github", "actions", "arduino"]
---

**This blog post was hosted in the Arduino Engineering Blog, you can
[read it here](https://blog.arduino.cc/2019/11/14/arduino-on-github-actions/).**

[GitHub Actions](https://github.com/features/actions) is the name of the SDLC
(Software Development Life Cycle) system introduced by GitHub about a year ago,
currently in public beta and now approaching the general availability status.
When you read SDLC, think about some sort of CI/CD system that’s generic enough
to let you define a sequence of operations not necessarily limited to build,
test, and deploy code: GitHub Actions can help you automate processes in code
reviews, issue triaging, and repository management.

GitHub Actions have been part of the tools we use at Arduino for a while now;
helping us solve a wide range of problems from automating workflows within our
backend infrastructure to managing the release process of our open source software.
In the spirit of giving back to the community, we started publishing some of the
Actions we developed internally so that they can be used right ahead from any
GitHub account that’s been granted access to the public beta of GitHub Actions,
and eventually in any GitHub repository.

Our Actions are available from [this repository](https://github.com/arduino/actions)
and there’s one in particular we think the Arduino community will be happy to
have – it’s called setup-arduino-cli and under the hood it takes all the steps
necessary to have the `arduino-cli` binary available in a Workflow.  This means
that in any step of your Workflow you can leverage the long list of features of
the Arduino CLI.

### Example

Let’s say you keep your sketches in a GitHub repository, and you want to be sure
that every time you push a git commit or you merge a pull request, the sketches
compile correctly on certain boards you own, for example a
[Nano 33 IoT](https://store.arduino.cc/arduino-nano-33-iot) and a
[Uno](https://store.arduino.cc/arduino-uno-rev3).
To keep the configuration at minimum, we can use a “build matrix”, so that
GitHub will start a different job for each one of the platforms we list in the
matrix, without the need to configure them explicitly.

You can find a working example here: https://github.com/arduino/arduino-cli-example.

```yaml
# This is the name of the workflow, visible on GitHub UI.
name: test

# Here we tell GitHub to run the workflow when a commit
# is pushed or a Pull Request is opened.
on: [push, pull_request]

# This is the list of jobs that will be run concurrently.
# Since we use a build matrix, the actual number of jobs
# started depends on how many configurations the matrix
# will produce.
jobs:
  # This is the name of the job - can be whatever.
  test-matrix:

    # Here we tell GitHub that the jobs must be determined
    # dynamically depending on a matrix configuration.
    strategy:
      matrix:
        # The matrix will produce one job for each configuration
        # parameter of type arduino-platform, in this case a
        # total of 2.
        arduino-platform: ["arduino:samd", "arduino:avr"]
        # This is usually optional but we need to statically define the
        # FQBN of the boards we want to test for each platform. In the
        # future the CLI might automatically detect and download the core
        # needed to compile against a certain FQBN, at that point the
        # following include section will be useless.
        include:
          # This works like this: when the platform is "arduino:samd", the
          # variable fqbn is set to "arduino:samd:nano_33_iot".
          - arduino-platform: "arduino:samd"
            fqbn: "arduino:samd:nano_33_iot"
          - arduino-platform: "arduino:avr"
            fqbn: "arduino:avr:uno"

    # This is the platform GitHub will use to run our workflow, we
    # pick Windows for no particular reason.
    runs-on: windows-latest

    # This is the list of steps this job will run.
    steps:
      # First of all, we clone the repo using the checkout action.
      - name: Checkout
        uses: actions/checkout@master

      # We use the arduino/setup-arduino-cli action to install and
      # configure the Arduino CLI on the system.
      - name: Setup Arduino CLI
        uses: arduino/setup-arduino-cli@v1.0.0

      # We then install the platform, which one will be determined
      # dynamically by the build matrix.
      - name: Install platform
        run: |
          arduino-cli core update-index
          arduino-cli core install ${{ matrix.arduino-platform }}

      # Finally, we compile the sketch, using the FQBN that was set
      # in the build matrix.
      - name: Compile Sketch
        run: arduino-cli compile --fqbn ${{ matrix.fqbn }} ./blink
```

You can find more info and docs for the Action on the
[GitHub Marketplace](https://github.com/marketplace/actions/setup-arduino-cli): if
you like it, please leave us a star!
