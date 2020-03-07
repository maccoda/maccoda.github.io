+++
date = 2020-03-07
title = "Don't fret over it not being perfect"
description = "In software we should be accept and utilize that we do not need to have our solution perfect before we can even write our first line of code"
draft = true
[taxonomies]
tags = ["iterative development", "development process"]
+++

Software is such a young and unique stream of engineering. Almost always we cannot physically see
the product that we produce as opposed to civil engineers for example, who have their buildings and
bridges to see and test. Along with this however comes a much higher cost and risk of their design
requiring to be right the first time round. On top of this many people will be using these final
products and a failure won't mean that they can't double tap to like some celebrities new fad diet.

Whilst this may sound like it is belittling the work we do as an industry, but the truth is this is
a super power that we have over every other stream of engineering. Once they have built and shipped
a car to the end customer, they cannot simply add a minor improvement to the brake pads and then
update all existing customers and still make a profit. We however can! This iterative development
cycle is one of the key factors of agile development and our ability to adapt to the change in
requirements. This is why we can pull out concepts like Kanban and Scrum, and then single out
the deficiencies of waterfall in our industry, something that works so well in others.

In my opinion one of the core pieces and that which I wanted to share some thoughts on is the
iterative versus upfront design. In particular my experience with this in a team working with Kanban
and how important it is reflect on your working process to avoid eventually grinding to a glacial
speed.
<!-- Maybe we should determine if we want to focus on gold plating or agile -->

## Iterative vs upfront design

One of the key aspects of Agile is in the name itself, have the agility to change how your product
develops. This is as opposed to the classic approach of completing all the design upfront before we
even build the first tiny piece. In this upfront design we will typically be inundated with
documentation and discussions on all possibilities. Again I emphasize that this has worked well in
numerous other industries but in the software industry I just don't believe it is a good fit. Doing
all this design upfront forces us to ask all the questions and have all the answers before we can
even start to test any of our hypotheses.

I have found that we can also use this mindset from the scale of an entire product, on a single
service, or even a single card from the backlog. This single card level is where I want to focus on
for the rest of this post.

## Iterative development with Kanban cards

Previously I had worked in a team following the Kanban approach, I won't go into details about this
style of working here, but the main point is we had our projects sliced up into smaller tasks which
we would pick up on to work on. It is these tasks or cards that I started to notice we were falling
into a lot of upfront design, gold plating, and what if scenarios. At first I thought this to be
good practice, designing for the worst and hoping for the best. It wasn't until after a few months
that I noticed this was starting to decay our work process, card scopes began to increase once work
had started, cards were starting to overlap with other cards making dependencies we didn't want, as
well when it came time to hand over we had to define a whole new set of definition of done.

My thoughts as to why this occurred was based on the fact we were prematurely gold plating our
solution, particularly since it was a new project were we were continually discovering new
requirements. My learning from this can be boiled down to the following topics:

- Watch the scope creep in cards
- Define and stick to acceptance criteria
- Utilize user stories to understand what you are achieving

### Watch the scope creep in cards

In our work process we had groomed all the cards prior to them being ready to be picked up, that
meant there was a pretty scope of what was required from the card, which had input from the
developers. However there will always come a time when you miss a subtle requirement or you find to
do this you need that. At this point it is very tempting to switch gears and take on the new found
task but you should avoid this as it then introduces new scope into your current card. Particularly
in a Kanban style of working where you are focussed on flow of work, increasing the scope of a card
is particularly problematic.

When you do find these new requirements or pre-requisites, don't view it as a fault in your process for not
knowing this beforehand but rather see if you can carve up a new card to capture these new
requirements. Doing it this way means that any dependencies on your current work won't get blocked
and it may even be able to be picked up by another team member immediately so it can be completed in
parallel.

If this new piece of work is a dependency on your current card it is still possible to split it into
a new card and not block yourself. Consider how you could put in a stub or something of the sort
which will allow you to progress and for that stub to be removed/implemented as part of the card you
have just created.

### Define and stick to an acceptance criteria

One of the great parts of the work process I had in this team was that we had a close working
relationship with QA. When we picked up a new card we defined an acceptance criteria or a definition
of done for that card. This was typically a bullet point list of some use cases QA could test
against and what they were to expect. What I ended up learning was that this was as useful to me as
a developer (if not more) as it was for QA. If I stuck to only implementing this acceptance criteria
then I could easily avoid over complicating the implementation. This meant that it was even more
important that I participate and understand where this acceptance criteria came from and really
focus on breaking down that barrier that we see all too often between developers and QA.

Only focusing on the acceptance criteria can mean that you know that there may be some
implementation detail you believe is missed. Again write this up into a new card because time and
time again I have found that I expected something was required only to find out later it is covered
by something else, requirements change, or I made incorrect assumptions. It more than likely is
better to write a card which captures your thoughts in the moment and reevaluate its importance
later on when you are ready to pick it up.

### Utilize user stories to understand what you are achieving

Finally when implementing a new piece of functionality it is always so much more satisfying if you
know the impact you will be having on customers. This is where user stories provide the answer.
Defining at least one user story for the card you are going to pick up feeds into a lot of positive
processes indirectly. Having a user story means that defining acceptance criteria will be simpler as
it is based on this user story. When you are considering the scope of work you need to complete
again this user story provides that basis on which you can check if your expected implementation
will handle that scenario.

Whilst I have mentioned the user story to a particular card sometimes this is difficult and one user
story may span several cards and hence the right place to create such a story is during the grooming
session.

The main aspect of gold plating this user story achieves is it helps us bring our attention to what
actual functionality we want to deliver to our customer and not all the ones that we think we want
to deliver.