+++
date = 2019-03-11
title = "Deploying Micronaut Application to Heroku"
description = "Deploying your hobby Micronaut projects"
[taxonomies]
tags = ["micronaut", "heroku", "deployment"]
+++

[Micronaut] is a great JVM framework I was recently introduced to. It has been
designed with the intention to fit into the modern day micro-service and
serverless architecture with its big selling point being compile time dependency
injection. If you have ever used something like Spring Boot you will know what a
difference this will make to start up times. However this post is about
deploying these apps to Heroku. Once I have had some more experience in the
framework I will write another post about making an application with it.
In lieu of that, I would really recommend giving it a try and reach out if you
have any blockers.

For this post I am assuming you are familiar with [Heroku] as I won't go in
depth about setting it up. However that is the beauty of this service is they
make it really simple to get started which is great for hobby projects.

## Creating the Application

Creating the app is very simple as with many frameworks. Micronaut provide a CLI
tool and a [guide] for creating your first application using this. Again I won't
go more in depth into this but it is very important to follow this [guide]
otherwise your experience may differ and likely be more difficult.

The important part of using the CLI tool is that is generates the base project
and along with that is provides the `Dockerfile` that can be used to build a
deployed image. It is a very simple `Dockerfile` but just highlights the
focus on simplicity in the modern environments provided by Micronaut.

## First Attempt to deploy to Heroku

My first attempt as the naming of this sections forbodes was not all that
successful and led to a few weird issues. Let's start at the beginning.

Choosing Heroku as the service to manage my deployments was primarily driven by
simplicity. To deploy my first app was as simple as `git push heroku master`
(after some minimal set up of course). This application should be no different I
thought, it is a Gradle application and Heroku natively supports with this a
Gradle build pack. Unfortunately not quite so, they know how to support Spring
applications but if none is detected you are in charge of telling the build pack
how to build your application. Which is a good choice and again is made simple
by enforcing a convention.

*If Heroku does not how to build your application you define a `stage` Gradle
task*

Of course I am not the first person to deploy a non-Spring application to Heroku
so there is a simple guide. For a Micronaut application the only required part
however is defining the `stage` task and its dependencies.

```groovy
task stage(dependsOn: ['build', 'clean'])
build.mustRunAfter clean
```

That was pretty simple!

Now if you were anything like me and just want to get your application out there
you would try to deploy this shortly after. In doing this I started to find
where most of my issues lay. After the deployment I attempted to check the
health endpoint and was receiving a fat nothing, my first thoughts were, 'Does
it know how to run my application?'

### Defining the Procfile

For those unfamiliar with Heroku, they have a notion of a `Procfile` which
defines which commands to run to start your application(s). Akin to that you
would write for a build pipeline.

First thing I thought here is I know I was able to run my application through
Gradle using the `run` command so let's first try that.

```yaml
web: ./gradlew run
```

Immediately I deployed again and tested the health endpoint. Same result...

### Finding the Error

It turned out the error was staring me in the face the whole time in the
deployment logs that I clearly do not pay the required attention to. When you
perform the git deployment you don't actually get all the logs of the container
you will need to use `heroku logs --tail` to get these, and they are incredibly
useful!

Littered throughout the logs I had occurrences of `Error R14 (Memory quota
exceeded)`. Not only that, it did not only appear during the execution of the
application but actually when trying to build the application!

Now of course I could purchase the plan providing me with more memory but this
was a hobby project. Further that is almost always the cheat's way out as there
is probably something better you can do before just upping allocated resources.

The reason for this issue I can only assume is because of the manner in which
Micronaut builds your application. A lot of the heavy lifting is done during
compile time giving you super fast start up times. However I was now pushing all
this load onto the Heroku containers for which I had limited resources for.

## Getting the Application Deployed

After realising I would need to perform the staging on my local machine I
started looking at different methods of deploying to Heroku. As it turns out
they provide you access to a container registry for which you can push images
and then release them for your application.

The process to deploy a container is as follows:

1. Define your deployable image
1. Build the artifacts needed for the image and consequently build the
   deployable image
1. Push the deployable image to the registry
1. Release the image to the deployed environment

This process is made extremely simple by Micronaut and Heroku! I was able to
script it in 3 lines!

```bash
./gradlew stage
heroku container:push web
heroku container:release web
```

These define the last 3 steps of the process above but where is our image
definition? It is the `Dockerfile` that Micronaut generated for us on the first
build!

```Dockerfile
FROM openjdk:8u171-alpine3.7
RUN apk --no-cache add curl
COPY build/libs/*-all.jar myapp-kt.jar
CMD java ${JAVA_OPTS} -jar myapp-kt.jar
```

All this `Dockerfile` does is simply copy the built artifact from the `stage`
task (which is combined into a single JAR ending in `-all`).

Finally we just need to update the `Procfile` as we do not need to build
anything anymore, but rather just need to simply execute the JAR. Of course be
aware there is a version appended.

```yaml
web: java -jar build/libs/myapp-kt-0.1-all.jar
```

Once you have updated the `Procfile`, deploy to Heroku again and you should find
your application up and running!

## Summary

To recap what we managed to achieve. We built our Micronaut application
following the provided guide. Then utilizing what was provided from the template
built a deployable Docker image which could then deploy to Heroku!

[Micronaut]: http://micronaut.io/
[Heroku]: https://www.heroku.com/
[guide]: https://guides.micronaut.io/index.html