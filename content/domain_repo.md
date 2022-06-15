+++
date = 2022-06-13
title = "Domain Mono Repos"
description = "A new way to think about how to structure your repositories"
+++

We use repositories every day as a software engineer. They are a simple
organisational tool to group together code for a certain project. Primarily
however they are driven by the fact they are a single repository in accordance with our
source control (git). However, as the list of tools and processes that hook into
these repositories grow, so too does the value of the repository. Slowly, it is
becoming coupled with many other elements of our work. For example, we have
plenty of tools to track history over a single repository using source control
management. We have deployment pipelines that are typically attached to a single
repository, which are responsible for getting our code out to production. We
also have issue tracking and project management that is attached to a single
repository. As we can see, the boundary at which we draw the repository is
actually having more and more of an effect on how we work than simply a basic
organisational tool.

My most common approach for repository structure has been to have a single
repository for a single deployable unit. Take a basic client server example.
Typically there would be a UI repository to manage the client side code, and an
API repository to manage the server side code. Along with that, if we are using
infrastructure as code (which we probably should be) there would even be a third
repository managing all the infrastructure. This model made a lot of sense
initially. Each of these elements are a different deployable unit and typically
there would only be a single language for each repository.

However, as projects grew, and we lean more into automation and tooling this
model started to show some cracks. To add value for the customer, it is uncommon
we can achieve this solely through the UI or the API. It is more common to need
to make changes across both these deployable units to add value. Therefore for a
single task we need to make changes to multiple repositories, which means
multiple code reviews. Not to mention needing to maintain more repositories
moving at different rates. This was obviously such as issue as the popular
editor VSCode even supports a notion of *workspaces* to work across multiple
repositories/directories. Once all parts were reviewed and integrated into the
pipeline then came the issue of ensuring the correct deployment order. It is
fairly common that the UI should depend on the API. Thus, we always require the
API to be deployed before the UI. This is not codified in our repositories as
most CI tools create a separate pipeline per repository, hence it becomes
difficult to ensure that the UI is never deployed without first having the API
with the relevant changes deployed.

These draw backs are what have led to the idea of a new repository style,
drawing on Domain Driven Design (DDD) and hence the name *Domain Mono Repos*.
The idea is that the same boundary that is created for a domain when designing
is the same boundary to define what sits within the domain mono repo. How this
looks in practice is that a single repository will contain all deployable units
and their infrastructure. For example, a single domain might consist of a NodeJS
API deployed on AWS ECS, a single page application UI and some AWS Lambda
Functions and AWS SQS queues. For this there would be a single repository that
separates these all into different directories, each having their respective
project structure.

Structuring a repository in this way addresses the issues mentioned earlier:
- a single feature can be implemented and raised in a single code review across
  all deployable units
- a pipeline can be created to ensure the correct deployment order for each
  deployable unit
- easier tracking of change history with commits being able to span the whole
  domain

Some other benefits that have been observed:
- logistically easier to find code and projects as there are fewer repositories
  and are named using the same language as the domain, hence easier to search
  for
- less template repositories to manage as they all essentially are created from
  the same start point
