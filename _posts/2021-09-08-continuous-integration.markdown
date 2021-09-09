---
author: Liam Björkman
tags: CI pipleine github actions
---



## What is a CI Pipeline?

A CI Pipeline is a streamlined and effective way of developing software in an agile way, especially when there's numerous people on your team working on the same project.



To understand why Continuous Integration is useful, it can be beneficial to understand what it replaces. Imagine being in a team of 30 developers, a lot of code is written, and sometimes there are multiple people working on the same file. Now this would not be a problem if all 30 people were constantly communicating with each other and presenting their changes to someone else's code. But not only does this require a huge portion of time, expecting everyone to always be available is unrealistic, and it does not hold up in an agile workflow. 

To overcome this and achieve agile workflow the team could instead develop against the same branch and constantly pulling/pushing changes made so that everyone is always up to date. That does increase the development speed for sure but introduces another problem; breaking changes. If the team constantly checks in code, there's no doubt going to be compile issues, bugs, etc...  

To solve this we implement some automation which is the core of the CI pipeline. A simple pipeline could consist of:

**the change -> building the application -> running tests**

If the pipeline is successful the changes did not break anything, but if it fails the developer will quickly know why and can fix it immediately while also maintaining developer flow. 

The image describes a simple CI pipeline:

<img src="/img/ci.jpg" width="400" height="400"/>



<br>

## Implementation of CI in SpacePark v1

I have implemented a simple CI Pipeline for my forked version of  the SpacePark v1 repo using GitHub Actions.

The pipeline will run on any push against the repo and thereafter build all projects for both Linux and Windows, since the project is based on .NET 5. If the build is successful it will continue with running the tests.

Details of how the pipeline is setup:

* **Setup job:** Basic environment is configured: OS, directories and all action repos are downloaded.
* **Run actions/checkout@v2:** Our first action is run which automatically checks out our repo and downloads it to the container.
* **Run actions/setup-dotnet@v1:** Downloads and installs the .NET SDK otherwise our project will not run.
* **Build Project:** Our projects are built using **`dotnet build`** which points at the `.csproj` file containing all projects dependencies. In our case it builds `SpacePark`, `ClassLibrary` and `SpaceTests`.
* **Run Space Tests:** After the build is successful, the pipeline will run `dotnet test` against `SpaceTests` 

* **Post Run actions/checkout@v2:** Cleans up files
* **Complete job:** Cleans up orphan processes and terminates the dotnet processes.

<img src="/img/gh.png"/>

<br>

## Workflow YAML File

```yaml
name: SpacePark Workflow
on: [push]
jobs:
    build:
        runs-on: $ {{ 'matrix.os' }}
        strategy:
            matrix:
                os: [ubuntu-latest, windows-latest]
        steps:
        - uses: actions/checkout@v2
        - uses: actions/setup-dotnet@v1
          with: 
            dotnet-version: 5.0.x
        - name: Build Project
          run: dotnet build ./Source/SpacePark/
        - name: Run Space Tests
          run: dotnet test ./Source/SpaceTests/
```

The workflow uses the matrix strategy which we can use if we want to run the same configuration but with different operating systems, platforms (different .NET versions, etc.) and languages for example.  The `matrix.os` is a special variable that will automatically select the correct OS from the array under based on the environment it is run in.

After that the steps are run parallelly in 2 environments, which we can see in the image above.

##### Sources

##### [Managing complex workflows](https://docs.github.com/en/actions/learn-github-actions/managing-complex-workflows)

##### [Developer Flow State and Its Impact On Productivity](https://stackoverflow.blog/2018/09/10/developer-flow-state-and-its-impact-on-productivity/)

##### [Continuous Integration](https://semaphoreci.com/community/tutorials/continuous-integration)

##### [The Eight Phases of A DevOps Pipeline](https://medium.com/taptuit/the-eight-phases-of-a-devops-pipeline-fda53ec9bba)

##### [CI/CD Pipeline](https://semaphoreci.com/blog/cicd-pipeline)

##### [GitHub Actions Syntax](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions)

