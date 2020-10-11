+++
date = 2020-10-11
title = "Introduction to Layered Architecture"
description = "An introduction to the layered architecture approach for organizing software projects"
[taxonomies]
tags = ["architecture"]
blog_series = ["software_concepts"]
+++

The inspiration of this post is I have worked with a few interns lately and the biggest sticking
point they have is from going from the problem solving in a single function as typical of most
computer science work to creating an application across multiple files and focusing on reusability
and adaptability to change. It is a big jump going from the mindset of pure computer science degree
and solving problems with the goal of correctness and optimization whereas once in the industry the
priorities shift to being able to change with changing requirements. With this article I intend to
show a simple first step by using the layered architecture.

When starting out on a new project the hardest thing to do is organise the architecture. What
packages/modules should I make? Where should I put this class? Should this piece of work be its own
class? A really safe place to start with is the layered architecture, there are plenty of articles
and books on this regarding if it is the best approach and in more depth. My goal from this article
is solely to help those working on one of their first projects be able to structure their code.

**What exactly is the layered architecture?**

The layered architecture as the name suggests splits your code into layers. Typically there are 3
for a web service, 4 if you have a UI. There is the controller layer which handles requests to your
service, the service or domain layer which has the business logic of your application, and finally
the repository or persistence layer. If you have a UI you will also want to have a presentation
layer.

Typically the rule of thumb here is that each layer can only have dependencies on the same level
layer or below. Again I want to stress though that there are different ways of doing this and
arguably better when mixed with this design approach but I have found this to be a great starting
design to make the initial progress. I can hopefully go over other architecture patterns in future
articles.

## Controller layer

In a typical web application this layer will be the one responsible for accepting the web requests.
This is where you want to do your input validation and then pass across the sanitized inputs to the
service/domain layer.

```kotlin
@Controller("/users")
class UserController(private val userService: UserService) {
  @Get("/{id}")
  fun getUser(id: Int): User {
    return userService.getUser(id) ?: build404Error()
  }

  @Post
  fun addUser(userDetails: UserDetails) {
    // Validation logic
    if (userDetails.name.isNullOrBlank() || userDetails.password.isNullOrBlank()) {
      return build400Error()
    }
    userService.addNewUser(userDetails)
  }
}
```

## Service/Domain layer

Depending on which crowds you chat with or which framework this layer may get a different name
however the purpose is the same. This layer contains the logic for the business logic of your
application. This is the more important part of your application as this is the section that makes
it do what it does. If you are making a bank application this is where you handle transactions and
checking balances, if it is a retail application you work with the shopping cart, etc.

The domain layer should not have any dependencies on any other layer. The main reason for this is
the other layers should ideally be able to be changed easily (although you would not want to do this
often) and have no impact on how the business logic works. I realize this is a little contradicting
to the first point I made about only knowing about layers on the same level and below but this is
where you realize that there are elements of software that are more of a craft than a hard science.
For this layer specifically what you want to avoid is it knowing anything about the infrastructure,
that is it should not know that it receives requests as a HTTP call, or it persists data in a
particular format using a certain SQL database. This should be abstracted away so the controller
should convert it to a domain object this layer can use and the persistance layer should have a high
enough abstraction in its API so this does not need to know of the logistics of how the data is
stored.

```kotlin
class UserService(private val userRepository: UserRepository) {
  fun addNewUser(userDetails: UserDetails) {
    val maybeExistingUser = userRepository.findWithName(userDetails.name)
    if (maybeExistingUser != null) throw UserAlreadyExists(userDetails.name)

    userRepository.save(userDetails)
  }
}
```

## Persistence layer

This is where you implement the logic for persisting the entities to the format used by your
application. This can be to a database, a flat file or in memory. As stated above you want this API
to be abstracted away from the actual implementation details. Doing this will allow you to quickly
get started and then allow you to change the persistence mechanism as the project grows, for example
I typically start with a flat file to get started and move to a database once (or if) it is needed.

```kotlin
class UserRepository(private val dataStore: DataStore) {
  fun save(userDetails: UserDetails) {
    val user = User(name = userDetails.name, password = Encryptor.encrypt(userDetails.password))
    dataStore.insert(user)
  }
}
```

## Presentation layer

This layer can take on many forms, it could be your sparkly new React frontend or could simply be a
HTML page served from your server. Regardless this layer needs to be separate and only have the
responsibility of taking the returned data and displaying it.

For this example I will not provide an example as there are many things it could be, it could be the
entire single page application that is making JSON API calls to the server and building the user
interface. Another alternative if it is server side rendered or simply a desktop application, is it
could be a template that receives the output of the controller layer and knows how to format and
arrange the data on the page.

## FAQ

### What should I do if I have to call another service or API?

When considering these classes it is the hexagonal architecture that starts to have a better fit
however I like to think of these at a similar location as the persistence layer as you typically
interact with a database in a similar fashion to another web service. A lot of databases nowadays
are just web APIs. The key part here is this should be separate from your domain logic.

### How should I split my domain layer?

There are many ways to slice your domain layer and the package, directory, or module structure has
so many varieties. One that I have begun to enjoy working with is feature/functionality based
splits. An example being for a banking application you can create a separate module for interacting
with bank accounts and a separate one for interacting with customers. There of course may be overlap
and I guarantee you that the functionally will not be as clear as the obvious examples I or others
may provide, so allow flexibility as you may not get it right the first time. The concept to
consider is that you should strive to be able to define a clear API into your module that defines a
use case of your application. The goal is that the module should only change for one business
reason.

**If I have missed a common question or one you would like answered please raise an issue on this
repository and I will gladly answer and update this post.**
