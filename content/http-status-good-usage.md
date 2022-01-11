+++
date = 2022-01-11
title = "Some good things to remember when using HTTP statuses"
description = "HTTP statues are a great tool that we can use to communicate to our clients the intention of our response. There is also a lot the client should know about them to use them correctly."
[taxonomies]
tags = ["http", "web development"]
+++

If you have worked on a client/server project of any sort then you have no doubt
encountered HTTP statuses. They are part of the HTTP standard and there are a
lot of them. I wanted to go through some useful elements to consider regarding
their usage as both the producer and consumer of a HTTP response.

First off, what are they exactly and what do they all mean? Thankfully we have
[MDN][mdn statuses] which has the entire list of them. As you can see they are
represented as a 3 digit number with ranging from 100-599 (currently at least).
The most important thing to note and the inspiration for writing this post is
that the statuses are grouped based on the first digit. This grouping is both
incredibly useful and incredibly important to recognise. It is useful because
you are able to easily categorise return types if you are not concerned with
specifics, such as for all errors show an error message. It is important because
I have seen many people write code that checks for a status of exactly 200 when
in fact the response could be any from the 2XX group.


## Authoring a server

When authoring a server which provides these statuses it is good to extend
beyond just the 200 and 500 options. In doing so it provides the client with
more information than simply success or failure, and therefore allowing the
client to make more informed decisions as to how to handle the response. The
following are two small examples where success and failure can be represented
with a different status code and thus would result in the client performing
different actions.

### 200 vs 204

Both the status of 200 and 204 represent a successful result but have slightly
separate meanings. 200 is **OK** and 204 is **No Content**. The commonplace 200
response will typically be accompanied by some body text and hence if this
response is received the client will likely attempt to parse the body element.
However if no such body element exists sending through a 204 will make the
client aware that no such body is present and can avoid attempting to parse it.

### 4XX vs 5XX

In terms of a failure scenario there are several ways in which this can be
categorised. Two ranges of errors are within the 4XX and 5XX range. When
constructing a server response it is important to consider which range is most
appropriate to assist the client in determining the next course of action. When
the server returns a 4XX response it means that the error is on the client side,
such as incorrect data or invalid credentials. In these cases there is no point
in the client retrying with the same input. Whereas if the failure was due to an
internal error (unrelated to client provided input) such as downstream
dependencies failures or unexpected exceptions, a 5XX code should be returned.
Depending on the exact response code the client can know whether a retry should
be performed or not.

## Authoring a client

Authoring a client using HTTP statuses is almost the opposite of authoring a
server. When authoring a client it is vital to not just assume there is the
success or failure case. Beyond that, the status alone can be used to obtain all
the information required.

### Do not assume a 200 for success

As mentioned in the server section, 200 is not the only status code for a
successful call. Therefore writing client code that explicitly checks to see if
the status is 200 will result in errors. For example, the status 201 is returned
if an entity has been created, such as a typical POST in a REST API. Therefore a
still successful scenario has now suddenly caused issues on the client side.


### Know the groupings

As mentioned previously, the HTTP statuses are grouped into 5 classes, 1XX to
5XX. Treating each of these individually allows for client code to be written to
respond differently depending on the class. One of the clearest examples is
provided above, knowing the difference between client errors (4xx) and server
errors (5xx). Using this information it is possible to write correct utility
methods around these to define common actions to be taken depending on the class
of HTTP status received.

### Utilise the status

Finally, an issue that I have observed is writing client code that just skips
the status check. The status classifies the type of response that has been
received and in some use cases this is sufficient so all other information can
be disregarded. A good example of this is a health check. A health check is a
simple HTTP call to ensure that a service is up and running, a client calling
this can determine the status of the service simply by looking at the status.
There may well be some body content here but there is no need to couple the code
to the contents of this response to determine the status of the service.

[mdn statuses]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Status
