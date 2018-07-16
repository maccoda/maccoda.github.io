+++
date = 2018-07-16
title = "How was I so wrong about CI and what I learnt about CD"
description = "Looking into exactly what is the whole notion of CI vs CD vs CD"
+++

My knowledge of Continuous Integration (CI) and Continuous Delivery (CD) had so
far only been developed from discussions with colleges and my own assumptions on
the topic. Only recently had I decided that I should actually put this knowledge
to the test and started doing some reading. To my surprise I was well off...

## Continuous Integration

First off I started looking into CI, you know the build machines and stuff yeah?
Nope. Continuous Integration is exactly as its name describes, continuously
integrating your code. For some unknown reason to me I had understood CI as the
build pipeline getting kicked off every time someone makes a commit to the
repository. In a way this isn't ridiculously off as this pipeline provides the
means for CI but is not it itself. After watching a [talk] by Martin Fowler he
described CI in such a raw, upfront manner that it has really stuck with me. I
paraphrase but he states something along the lines of;

*If you can answer yes to the following 3 questions, then indeed you **are**
practicing continuous integration:*

1. *Are all engineers pushing to trunk/master on a daily basis?*
1. *Does every commit run a solid suite of tests giving confidence the commit works?*
1. *When the build is broken, is it typically fixed in under 10 minutes?*

This is pretty far from a build pipeline tool right?

So what we are saying here is that CI is the practice of continually integrating
developer's code in the repository, **daily**. That is, there is no never ending
feature branches and no 'sub-master' branch that we all merge into before we go
to our actual master, nope. Everything needs to be integrated and on the master
branch. Period. I have definitely witnessed teams not practicing CI and seeing
how things can gradually diverge over time and then when finally ready, bringing
them all back together becomes feature work in of itself!

Next up, we are saying, cool you are all integrating but does the software still
do what it is meant to? The easiest way to get peace of mind for this is a solid
suite of tests. Now we are all well familiar with unit tests but through my
readings I was introduced to so many different types of tests, all of which play
a different role in building our confidence in our software. If you are
interested in looking further into it have a read of [this article] by
Atlassian. So what we really want here is a suite of tests, not only unit tests,
that is kicked off every time some one delivers and definitely every time some
on commits to the master branch.

The final question is directed more at the culture developed, than the actual
practice. I feel that it is depicting the importance of the master branch. The
notion of, if it fails, you drop everything and get that baby back to green! To
achieve this there has to be a culture shift, as well as confidence in your
build pipeline. The last thing you want is developers not thinking twice about a
failed build because 'everyone knows this test is flaky'. If it is flaky and you
can do something about it, fix that up, regain some confidence in your pipeline
and get that pipeline to a strong green.

[this article]: https://www.atlassian.com/continuous-delivery/different-types-of-software-testing

## Continuous Delivery vs Continuous Deployment

The two CD's were a lot newer to my vocabulary than the wrongly understood CI,
so I approached these with less of a tainted mind. Again Martin Fowler covered
these topics in his [talk]. There is so much to talk about regarding these
topics and it is really fascinating if you get the chance to read further into
them but here I just wanted to look at *what exactly is the difference?*

Firstly I want to say that I am by no means an expert in regards to these but
just want to share what I thought was a really simple explanation on
differentiating the two. Both of these notions revolve around very similar
concepts and both have similar end goals. They are looking at being able to
get quality software out the door in an automated, reproducible manner, and of
course, quickly. One of the biggest pre-requisites is to have an automated
pipeline, script where you can, define your pipeline in source code, get
everything moving with as little human interaction as possible! Once this is
done we can look into getting the software into environments of closer parity to
the production environment than our local development machine, again ideally in
an automated manner. This is all so we can gain confidence that the release will
work as well as be certain that it is done in the same manner every time.

So at this point we haven't really said much about what is the actual difference
between delivery and deployment. Well that is because up until now there kind of
isn't much of a difference. However when we get to the final stage delivering
the software into production that is where it differs. **Continuous Delivery**
is having the ability to deliver the current master at any given moment,
**Continuous Deployment** is actually doing just that for every commit. You can
see how the two can be easily confused, further how continuous delivery is sort
of a requirement for continuous deployment.

Actually implementing these practices can have some unexpected side effects as I
have seen. Applying only continuous delivery, teams had no problem practicing
continuous integration as they had their master branch raring and ready, but
only deployed when it was required. Whereas those practicing continuous
deployment actually gravitated away from the continuous integration definition
given earlier as they started having these sub-master branches so that they
could be confident in their software before it was delivered to production.
However I am sure this all depends on the scenario and what your team has in
place, I am certainly not saying continuous deployment is bad as I am sure the
big tech companies have definitely made this work well for them. Give it a go
and see what works best for your team.

[talk]: https://www.youtube.com/watch?v=aoMfbgF2D_4