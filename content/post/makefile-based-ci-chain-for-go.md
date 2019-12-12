---
title: "A Makefile based CI/CD chain for Go"
description: "Building a Makefile for Go"
date:   2018-04-19T16:00:00+02:00
author: Sebastien Le Gall
categories: [Go, Makefile, travis]
---

![lunch button](/img/1_yUdrAB9i_f8oZE771sTSPA.png)

Whatever the project you are working on, whatever the programming language, using a Makefile  to handle common tasks is a great idea.

Makefiles are great at :

* Mask the complexity of a task;
* Handle multiple tasks;
* Help contributors to start with your project;
* Manage configuration using env var;
* Define a common interface / standard between projects;
* Abstract the underlying technology.

For all those reasons, `make` is a perfect fit with any CI/CD chain. Using a Makefile helps you define a simple and common interface usable whatever the CI/CD chain you plan to use. The only thing you will have to worry about is that your CI/CD chain tool will call the right **target**.

```sh
make something
```

And the job is done. It could be done with Travis. It could be done with Jenkins. The only thing that matters is the interface defined by your targets.

![makefile](/img/makefile-logo-selected_1x.png)

<!--more-->

## Readability

![readbility](/img/5-Quick-Ways-To-Improve-Your-Website-Usability-Readability.jpg)

Makefiles help you make things more readable than a `.sh` script file where people tends to add lot of unnecessary complexity (colorized output, comment, etc.).

If your Go project require `dep` to be built, you may want to add a `build.sh` file where you could put something like : 

```sh
go get -v -u github.com/golang/dep/cmd/dep
dep ensure
go build
```

This way, building the project is really easy for new contributors : 

```sh
./build.sh
```

But then, it means that every time you want to build a project, you will do an unnecessary `go get`.

Other solutions could be :

* Adding a `if` statement in your `build.sh`, checking for dep existence. *This option will increase the complexity of your script and make it less readable*;
* Adding a `dep.sh` that make the `go get` once for all. *But then you will have to maintain another script*;
* Do nothing and just make sure `dep` is listed as a requirements in your `README`. *But what if there are other requirements? Like gox ? Or goveralls?*.

Using a Makefile here could be a good idea. We will see how.

## Predictability

![predictability](/img/istock-499568538.jpg)

Makefiles can also be used as a common interface between many projects / libs. Particularly if you are working with many microservices.

> Looking at code you wrote more than two weeks ago is like looking at code you are seeing for the first time. - Dan Hurvitz

When switching between several projects, it may be painful to open the readme file just to check how tests are supposed to be launched (assuming tests are documented, which is mostly not the case…) just because you can't remember what strategy you chose for tests on this particular project.

Yet, you could use a shell script file. But what if you want to check if either or not the `test.sh` file also launch code coverage? Your script file may be simple and readable. But why doing such a script if it is so simple?

You may be tented merge the build and test scripts together. But then, you will have a "multi-task" script. 

There is a tool for that : Make.

## Reproducibility

![Reproducibility](/img/Replication.jpg)

Managing multiple CI/CD tools such as Travis, Gitlab CI, Circle CI or Jenkins may be painful. Each have an *same-but-not-the-same* way of doing things. They all use `.tool-name.yml` yaml configuration file where you can configure the way to run "pre-build" job, "build" job, etc. But each also have its own way to handle build stage and configuration keys.

Using Makefiles help you define a list of common target and use them on all your projects.

The same way, it became a convention to have a `npm start` command in the JS world, you could have at least 2 common targets to use on all your projects : `make build` and `make test`.

This way, any CI/CD job will look the same, whatever the project and the technology :

1. Build the project using `make build`
2. Run tests using `make test`

## Building a Makefile for Go

The following example of Makefile is designed for a Go application. For the purpose of this blog post I will assume our Go code is hosted on Github. We will use Travis as a CI / CD chain but you will see that the CI/CD provider does not matter. 

Concerning the deployment, we will create multiple binaries built to be share trough Github releases in order to be downloadable easily by users. 

We will assume our Go code is organized following this structure : 

```sh
.
├── CONTRIBUTING.MD
├── Gopkg.lock
├── Gopkg.toml
├── LICENSE
├── Makefile # Our makefile
├── README.md
├── bin # where binaries will placed at compile time
├── cmd # where application command related code takes place.
├── main.go # Go entrypoint file
├── pkg # where internal packages take place
├── scripts #we will use 1 script to handle code coverage
└── vendor
```

Let's assume our CI/CD chain will handle at least those jobs : 

1. Build the application
2. Run tests
3. Publish code coverage
4. Publish binaries as Github release (deployment)

### Opinionated dependencies

#### Dependency management

External dependencies used to be [a huge issue in the Go world](https://blog.gopheracademy.com/advent-2016/saga-go-dependency-management/) . `dep` is a response to that issue. I am a big fan of `dep` and use it on all my projects since `dep` not just manage dependencies. It also is capable of retrieving dependencies from your code and from other dependencies management tools such as `Glide` and `Godep`.

Because `dep` is also one of the few dependencies management tools actually developed and maintained by the language core contributor, I highly recommend to use it as often as possible.

#### Cross-compilation

One of the strength of Go is its ability to do cross-compilation. Concerning the build and the deployment of our application we could easily use the `go build` command. But `gox` has this *"little thing"* that makes it great to handle cross-complication : It can parallelize builds.

The result is a faster cross-compilation. We will use it to build our binaries.

#### Code coverage

I'm not a big fan of defining code coverage as a code quality metric. I largely prefer having a few smart tests than a huge amount of tests that  cover every line of my code but are not relevant.

However, code coverage may be helpful when reviewing PR (your own ones or some made by external contributors). It can help spot irrelevant tests.

Using an external service publishing coverage on a PR make things easier than having to check the build logs. [coveralls.io](https://coveralls.io/) does the job pretty well and it is already compatible with travis-ci.

However, using coveralls.io means being able to send coverage report to the service. To do that, we will use the `github.com/mattn/goveralls` lib. It is a simple Go client for Coveralls.

We will see later that sending coverage report is not so easy. We will need a shell script to handle this part. This script will however be called from our Makefile to keep the readability and the predictability. This way, if you decide later to change the way to publish coverage report, the interface will remain the same.

### Configuration and phony

Let's start building our Makefile with some common and specific configuration. When working with Makefiles, it is generally a good idea to handle configuration thought environment variable set at the beginning of the file. This brings more readability of the targets and makes things easier to change if needed. 

In our example we will use `gox` to build binaries for different OS / arch:

* Linux
* MacOs
* Windows

We also need to set some common Make configurations. 

Here is the result :

```makefile
SHELL := $(shell which bash) # set default shell
# OS / Arch we will build our binaries for
OSARCH := "linux/amd64 linux/386 windows/amd64 windows/386 darwin/amd64 darwin/386"
ENV = /usr/bin/env

.SHELLFLAGS = -c # Run commands in a -c flag 

.SILENT: ;               # no need for @
.ONESHELL: ;             # recipes execute in same shell
.NOTPARALLEL: ;          # wait for this target to finish
.EXPORT_ALL_VARIABLES: ; # send all vars to shell

.PHONY: all # All targets are accessible for user
.DEFAULT: help # Running Make will run the help target

```

### Handling dependencies

Whatever the CI/CD tool you choose to use, dependencies described above should be clearly exposed and handled in your project. You should never presume the developer or the CI/CD VM or container already have those dependencies installed. 

Instead, make a target in your Makefile that handle requirements.

```makefile
dep: ## Get build dependencies
    go get -v -u github.com/golang/dep/cmd/dep && \
    go get -v -u github.com/mitchellh/gox && \
    go get -v -u github.com/mattn/goveralls
```

### Building the app

Building a Go app is very simple and can be resume in 2 words : `go build`.  But since we are using `dep`, we also want the dependencies to be retrieved. 

```makefile
build: ## Build the app
    dep ensure && go build
```

### Cross-building

As written before, we will use `gox` to handle cross-compilation. We also want our binaries to be stored in the `bin/` directory. OS and Arch are set in env var, we will make use of that in our command : 

```makefile
cross-build: ## Build the app for multiple os/arch
    gox -osarch=$(OSARCH) -output "bin/blackbeard_{{.OS}}_{{.Arch}}"
```

### Running tests

Since tests are part of the standard lib in go, running tests is easy :

```makefile
test: ## Launch tests
    go test -v ./...
```

### Building code coverage report

As previously written, building and sending coverage report to coveralls.io implies a few more logic. We could write this logic in our Makefile under the  `test-cover` target. But it would make things more complex to read. Remember : our Makefile is an interface where new developers (could be you) can understand as quickly as possible all the needed actions to do to start work with your code.

Let's create a `scripts/test-coverage.sh` file where we will put our coverage reporting logic : 

```sh
#!/usr/bin/env bash
echo "mode: set" > acc.out
for Dir in $(find ./* -maxdepth 10 -type d | grep -v vendor);
do
        if ls $Dir/*.go &> /dev/null;
        then
            echo "Testing $Dir"
            go test -v -coverprofile=profile.out $Dir
            if [ -f profile.out ]
            then
                cat profile.out | grep -v "mode: set" >> acc.out
            fi
fi
done
goveralls -coverprofile=profile.out -service travis-ci -repotoken $COVERALLS_TOKEN
rm -rf ./profile.out
rm -rf ./acc.out

```

Then, inside our Makefile we only need to create a `test-cover` target calling this script : 

```makefile
test-cover: ## Launch tests coverage and send it to coveralls
    $(ENV) ./scripts/test-coverage.sh
```

### Providing help

Even if providing help is not mandatory, it is something I really like to implement in a Makefile. It helps the user (and more often, me) knowing what to do when working on the project. 

To do that, I use the `MAKEFILE_LIST` env var provided by Make to get the list of defined target.

Did you notice the use of those "##" right before the comments I have put aside each target? This is not a mistake. Those "##" will help me extract comments I want to show the user when he will try `make help` : 

```makefile
help: ## Show Help
    grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'
```

This way, the user will be able to understand very quickly what to do with this project by typing `make`.

Cherry on the cake : by doing this, your Makefile become self-documented !

### Putting all together

At the end, our Makefile will look like :

```makefile
SHELL := $(shell which bash) # set default shell
# OS / Arch we will build our binaries for
OSARCH := "linux/amd64 linux/386 windows/amd64 windows/386 darwin/amd64 darwin/386"
ENV = /usr/bin/env

.SHELLFLAGS = -c # Run commands in a -c flag 
.SILENT: ; # no need for @
.ONESHELL: ; # recipes execute in same shell
.NOTPARALLEL: ; # wait for this target to finish
.EXPORT_ALL_VARIABLES: ; # send all vars to shell

.PHONY: all # All targets are accessible for user
.DEFAULT: help # Running Make will run the help target

help: ## Show Help
    @grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'

dep: ## Get build dependencies
    go get -v -u github.com/golang/dep/cmd/dep && \
    go get github.com/mitchellh/gox && \
    go get github.com/mattn/goveralls

build: ## Build the app
    dep ensure && go build

cross-build: ## Build the app for multiple os/arch
    gox -osarch=$(OSARCH) -output "bin/blackbeard_{{.OS}}_{{.Arch}}"

test: ## Launch tests
    go test -v ./…

test-cover: ## Launch tests coverage and send it to coverall
    $(ENV) ./scripts/test-coverage.sh
```

Any user will then be able to know what is possible to do with this project by using a simple `make` command :

```sh
$ make
help                           Show Help
dep                            Get build dependencies
build                          Build the app
cross-build                    Build the app for multiple os/arch
test                           Launch tests
test-cover                     Launch tests coverage and send it to coverall
```

## Setting up the CI/CD chain.

Once our Makefile is ready, the only thing left is to tell Travis to use our Make target on each build stage : 

```yaml
language: go
go:
  - 1.8
  - 1.9
  - master
before_install:
  make dep
install:
  make build
script:
  make test-cover
after_success:
  make cross-build
  
deploy:
  provider: releases
  file_glob: true
  api_key:
    secure: xxx
  file: bin/*
  skip_cleanup: true
  on:
    tags: true
```

This configuration will ensure the Travis VM has all our dependencies and will push Github releases on each tag pushed on master.

## Conclusion

Except for the last part, dedicated to Travis example, nothing in our Makefile is depending neither on the CI/CD chain nor the dependencies used to build our app.

For the purpose of this blog post, I chose to keep things simple and rely on Travis because Travis integrate Go. I could also have chose a CI/CD chain that does not support Go natively. The only thing I would have to do is to change the content of my targets to use Docker. Instead of installing dependencies like `dep` I could have a pre-built Docker container where I know `dep` in available.

The logic works also for compilation, etc.