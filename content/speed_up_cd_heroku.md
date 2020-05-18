+++
date = 2020-05-09
title = "Speed up your Gitlab pipelines to Heroku"
description = "Some useful techniques to improve or create a CD pipeline for Heroku deployment from Gitlab"
[taxonomies]
tags = ["gitlab", "continuous delivery", "continuous deployment", "heroku"]
+++

Arguably one of the best things one can have for their project is a robust continuous delivery
pipeline. Being able to know that once you commit and push your code there is infrastructure in
place to ensure your change is correct and subsequently deployed is one the things I personally
truly enjoy in software. So when I had some spare time I started tinkering with one of my side
projects in getting it set up and deployed to Heroku. My project is hosted at [Gitlab] which have
their own integrated CI service and is deployed to [Heroku] a platform as a service (PaaS) provider
which manages a lot of the deployment infrastructure for me, perfect for a side project.

For this project I had set up the following stages:

<div class="mermaid">
graph LR
  t[Test]
  pp[Deploy Pre-Prod]
  it[Integration Tests]
  p[Deploy Prod]
  t --> pp
  pp --> it
  it --> p
</div>

Definitely a little more involved than most of my side projects but hopefully it actually closer
represents would you would typically encounter for production services in the industry.

The application I had was a [Kotlin] web server application built with [Gradle] and deployed to
Heroku using a very versatile deployment tool [dpl]. All of this definitely worked and did what it
needed to however it took around 17 minutes and there are always things we can improve!

## Using a cache

Gitlab CI runners are declaratively defined and based on [Docker] containers. Each job executed in
the pipeline you will get an entirely fresh environment which is great to provide a reproducible
environment but this does require the environment to be recreated every time. In the case of Gradle
and the Gradle wrapper this means that on each stage of the pipeline that was using a Gradle task it
would need to download the distribution.

![gradle download](/images/gradle-download.png)

To avoid needing to download this at each job we can simply add in a [cache] in our pipeline. This
cache is saved at the end of each job and restored at the start, so it does come with its own cost
but compared to downloading Gradle and all project dependencies it is minor. To achieve this for my
Gradle project I simply changed the directory for where Gradle saved the dependencies and told
Gitlab CI to cache it, with a lot of help from [this StackOverflow question][so].

```yaml
before_script:
  - export GRADLE_USER_HOME=`pwd`/.gradle

cache:
  paths:
    - .gradle/wrapper
    - .gradle/caches
```

Do note that with this configuration your cache will be persisted and restored across all jobs
(assuming you have not placed this in a single job already) so if not all stages require this cache
we could adjust this to shave off some extra time but for now it will do just fine.

## Deploying with Docker to Heroku directly

The change that gave me the biggest improvement was by converting my deployment from using the [dpl]
tool to using the [container registry] Heroku provides.

The speed of deployment is less attributed to the dpl tool and more towards how Heroku defaults its
deployments. If you look across all of the Heroku website it shows how simple it is to perform
deployments with git. This is definitely a positive for simplicity of deployment but in doing so it
means that each deployment needs to be built from source each time. Using the registry we are able
to create an immutable, deployable Docker image that we can reuse at each stage of deployment.

First thing you will require is that your application builds to a Docker image, which I will not
delve into here (maybe another post). My application was actually already being deployed through
Docker with dpl by using the [heroku.yml] file, so my Docker image was ready to go.

With a deployable Docker image ready the next step is to add a build and publish step to our
pipeline. For those not familiar with publishing Docker images, you can consider Docker images akin
to a release of a library/dependency. These are published to some central registry with a specific
identifier and version, the same is done for Docker images. We can publish a specific application
and version to a registry, here it is hosted by Heroku others include [ECR (AWS)][ECR] or of course
[DockerHub], which is then fetched and deployed. Coming back now let's add this build and publish
step to our pipeline, firstly lets define some variables:

```yaml
variables:
  CONTAINER_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG
  PREPROD_APP_NAME: my-pre-prod-app
  PROD_APP_NAME: my-prod-app
```

These variables will simply make it easy to reference the identifier given to our container image
using some [variables][gitlab-variables] that Gitlab will populate for us as well as the names of
our Heroku applications. With these in place we can now create a build step, I have added this to my
test stage so the tests and build can run in parallel.

```yaml
build:
  image: docker:19.03.1
  services:
    - docker:19.03.1-dind
  stage: test
  variables:
    HEROKU_PREPROD_IMAGE: registry.heroku.com/${PREPROD_APP_NAME}/web
    HEROKU_PROD_IMAGE: registry.heroku.com/${PROD_APP_NAME}/web
  script:
    - docker login --username=_ --password=$HEROKU_API_KEY registry.heroku.com
    - docker build -t $CONTAINER_TEST_IMAGE .
    - docker tag $CONTAINER_TEST_IMAGE $HEROKU_PREPROD_IMAGE
    - docker push $HEROKU_PREPROD_IMAGE
    - docker tag $CONTAINER_TEST_IMAGE $HEROKU_PROD_IMAGE
    - docker push $HEROKU_PROD_IMAGE
```

The above follows from the example provided in the [Gitlab documentation][container-registry] with
some tweaks to target the Heroku container registry instead of the Gitlab one. In Heroku each
application has its own container registry, since each stage is its own application we need to push
the built image to both registries.

The one new variable here is the `HEROKU_API_KEY` which I have injected into the pipeline. This is
the API token that is generated by Heroku to allow access to the API, this can be accessed in your
Heroku account settings.

With the image built and pushed to the Heroku container repository we can now move onto the final
stage which is the actual deployment to Heroku.

```yaml
preprod:
  stage: preprod
  script:
    - apt-get update -qy
    - apt-get install -y curl bash
    - curl https://cli-assets.heroku.com/install.sh | sh
    - heroku container:release -a ${PREPROD_APP_NAME} web
  only:
    - master
```

The only way to perform the deployment is through the Heroku CLI therefore the first step here
(after obtaining dependencies) is to download the CLI. We can then simply perform the
[container:release] operation which will deploy the most recent version of our built application
from the container registry. The same step can be done for the production stage by simply changing
the app name variable.

Using this technique over `dpl` the deployment time went down from around 5 minutes to around 1.5
minutes for each deployment! Reducing this deployment time is extremely valuable to allow for the
ability to roll forward with continuous deployment pipelines, as well as a great excuse to learn
something new!

## Completed file

The completed pipeline file looks something like this:

```yaml
stages:
  - test
  - preprod
  - integration_test
  - prod

before_script:
  - export GRADLE_USER_HOME=`pwd`/.gradle

cache:
  paths:
    - .gradle/wrapper
    - .gradle/caches

variables:
  CONTAINER_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG
  PREPROD_APP_NAME: my-pre-prod-app
  PROD_APP_NAME: my-prod-app

build:
  image: docker:19.03.1
  services:
    - docker:19.03.1-dind
  stage: test
  variables:
    HEROKU_PREPROD_IMAGE: registry.heroku.com/${PREPROD_APP_NAME}/web
    HEROKU_PROD_IMAGE: registry.heroku.com/${PROD_APP_NAME}/web
  script:
    - docker login --username=_ --password=$HEROKU_API_KEY registry.heroku.com
    - docker build -t $CONTAINER_TEST_IMAGE .
    - docker tag $CONTAINER_TEST_IMAGE $HEROKU_PREPROD_IMAGE
    - docker push $HEROKU_PREPROD_IMAGE
    - docker tag $CONTAINER_TEST_IMAGE $HEROKU_PROD_IMAGE
    - docker push $HEROKU_PROD_IMAGE

test:
  stage: test
  # Test step...

preprod:
  stage: preprod
  script:
    - apt-get update -qy
    - apt-get install -y curl bash
    - curl https://cli-assets.heroku.com/install.sh | sh
    - heroku container:release -a ${PREPROD_APP_NAME} web
  only:
    - master

integration:
  stage: "integration_test"
  # Integration step...

production:
  stage: prod
  script:
    - apt-get update -qy
    - apt-get install -y curl bash
    - curl https://cli-assets.heroku.com/install.sh | sh
    - heroku container:release -a ${PROD_APP_NAME} web
```

[Gitlab]: www.gitlab.com
[Heroku]: www.heroku.com
[Kotlin]: www.kotlinlang.org
[Gradle]: www.gradle.org
[dpl]: https://github.com/travis-ci/dpl
[Docker]: https://www.docker.com/
[cache]: https://docs.gitlab.com/ee/ci/caching/
[so]: https://stackoverflow.com/questions/34162120/gitlab-ci-gradle-dependency-cache
[container registry]: https://devcenter.heroku.com/articles/container-registry-and-runtime
[heroku.yml]: https://devcenter.heroku.com/articles/build-docker-images-heroku-yml
[ECR]: https://aws.amazon.com/ecr/
[DockerHub]: https://hub.docker.com/
[gitlab-variables]: https://docs.gitlab.com/ee/ci/variables/predefined_variables.html
[container-registry]: https://docs.gitlab.com/ee/user/packages/container_registry/#container-registry-examples-with-gitlab-cicd
[container:release]: https://devcenter.heroku.com/articles/heroku-cli-commands#heroku-container-release
