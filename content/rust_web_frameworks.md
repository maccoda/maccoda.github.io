+++
date = 2018-05-01
title = "Making microservices in Rust"
description = "Trying out some of the frameworks currently available in Rust toput together a microservice"
+++

This is a small idea that I have been wanting to put together for quite some
time now and finally have managed to get the time and most importantly
experience in Rust to finally try something a little more than just small
projects. One area that I think Rust is really making a decent headway in in the
web domain, which I am assuming is likely due to its origin from Firefox. So I
wanted to see if I could put together a really basic CRUD micro-service doing
the ever so original TODO functionality.

My main goal from this project was to be able to try some larger scale crates
available in the Rust ecosystem and perform some more typical enterprise
activities of working with databases and HTTP requests in Rust, something I have
usually been steering clear of. Further to this, I was really hoping that I
could help some of these frameworks grow with some more examples of how to put
them all together so hopefully can make it easier to pick up for new comers.

If you aren't up for the read and would just like the code go check out the [repository][repo].

So what were the frameworks that I wanted to explore?

## Diesel

[Diesel][diesel] is one of the more popular [ORM][orm] in the Rust community. It has some
great documentation to get set up and to understand how to use it. It also comes
with a very handy CLI tool to assist in the setting up of a project and managing
database migrations, which is great for keeping everything nice and uniform and
automated. A great combination indeed!

The [getting started guide](http://diesel.rs/guides/getting-started/) for Diesel
provides a solid foundation on how to put it all together so I won't try rewrite
that, rather provide a light overview of the steps to be able to reproduce.

### Setting up the migrations

Following the guide the CLI tool will create an `up.sql` and `down.sql` in the
`migrations` directory to handle our initial database migration.

```shell
$ diesel migration generate tasks
```

This will generate the files and place them into a time stamped directory,
appended with the name of the migration we gave, which was `tasks`. In here we
want to put the SQL query to create our table and the opposing query to undo the
change.

`up.sql`

```sql
CREATE TABLE tasks (
  id SERIAL PRIMARY KEY,
  task VARCHAR NOT NULL,
  completed BOOLEAN NOT NULL DEFAULT 'f'
)
```

`down.sql`

```sql
DROP TABLE tasks
```

### Adding the Models

Following the remainder of the guide we get to develop the model of our Tasks
which then allows us to use the CLI tool to generate our schema.

We want to follow the recommended guide of having one type to represent the
structure we want to query from the database and one type to represent the
structure to insert into the database. These will be `Task` and `NewTask`
respectively and will be put in the `model` module.

```rust
#[derive(Queryable)]
pub struct Task {
    id: i32,
    task: String,
    completed: bool,
}

#[derive(Insertable)]
#[table_name = "tasks"]
pub struct NewTask {
    task: String,
    completed: bool,
}
```

### Generating the Schema

Once we have set up the models we can then go back again to the Diesel CLI to
generate the schema for us which will provide the DSL that we will interact
with.

```shell
$ diesel print-schema > src/schema.rs
```

This will look at our source code and locate the `struct`s that are `Queryable`
to determine the shape of our data. The resulting `schema.rs` should look like
the following:

```rust
table! {
    tasks (id) {
        id -> Integer,
        task -> Varchar,
        completed -> Bool,
    }
}
```

### Adding the Functionality

The final part to the database aspect of our micro-service is to provide some
functionality on interacting with the database with the CRUD operations. My
implementation is a very basic one to allow me to associate these interactions
with the model type.

```rust
impl Task {
    pub fn all(conn: &PgConnection) -> Vec<Task> {
        use schema::tasks::dsl::*;
        tasks.load::<Task>(conn).expect("Could not load tasks")
    }

    pub fn create(conn: &PgConnection, task: NewTask) {
        use schema::tasks;
        diesel::insert_into(tasks::table)
            .values(&task)
            .execute(conn)
            .expect("Unable to insert");
    }

    pub fn update(conn: &PgConnection, task_id: i32, task_update: Task) -> i32 {
        use schema::tasks::dsl::*;
        diesel::update(tasks.find(task_id))
            .set((
                task.eq(task_update.task),
                completed.eq(task_update.completed),
            ))
            .execute(conn)
            .expect("Failed to update");
        task_update.id
    }

    pub fn delete(conn: &PgConnection, task_id: i32) {
        use schema::tasks::dsl::*;
        diesel::delete(tasks.find(task_id))
            .execute(conn)
            .expect("Failed to delete");
    }
}
```

The above code shows a few different ways in which we can use the generated DSL
to work with our data, or in the case of the `create` function, how we can use
it without the DSL.

Note that it is important that the DSL is only imported on the function scope
and that when doing this ensure that the function parameters don't match any
column in the schema to avoid any unexpected shadowing.

[orm]: https://en.wikipedia.org/wiki/Object-relational_mapping
[diesel]: https://diesel.rs/

## Rocket

[Rocket][rocket] is quite a popular web framework, for several reasons but one
of course is the well polished website! It is pretty easy to get started with
and handles a lot of the boiler plate through its powerful use of macros. This
part of the project I cannot claim I developed a lot for as it does have a great
[tutorial][rocket-tutorial] on the website for how to use Rocket with Diesel. So
I will just provide some basic changes to this that I ended up applying.

The first thing being that I wanted to use PostgreSQL which wasn't covered in
the tutorial. This was a very minor deviation from the tutorial however because
it was mainly handled in the Diesel section of the code which was already
described previously. The biggest change here was changing the features that
Diesel was compiled with:

`Cargo.toml`

```toml
diesel = { version = "1.0.0", features = ["postgres"] }
```

The only other chunk related to this was the database connection pool that we used
which was simply changed to a `PgConnection`.

```rust
type Pool = r2d2::Pool<ConnectionManager<PgConnection>>;
```

### Managing the Database Connection

From there I was able to follow the tutorial fairly closely. We start by
ensuring that Rocket manages our database connection pool. This is as simple as
adding it to `ignite` function.

```rust
type Pool = r2d2::Pool<ConnectionManager<PgConnection>>;
fn init_pool() -> Pool {
    dotenv::dotenv().ok();
    let database_url = env::var("DATABASE_URL").expect("DATABASE_URL must be set");
    let manager = ConnectionManager::<PgConnection>::new(database_url);
    r2d2::Pool::new(manager).expect("db pool")
}

pub fn start() {
    rocket::ignite()
        .manage(init_pool())
        .launch();
}
```

Now that Rocket is managing our state we want to be able to grab that from our
requests. To do this in Rocket we need to implement the `FromRequest` trait for
our database connection.

```rust
impl<'a, 'r> FromRequest<'a, 'r> for DbConn {
    type Error = ();

    fn from_request(request: &'a Request<'r>) -> request::Outcome<DbConn, ()> {
        let pool = request.guard::<State<Pool>>()?;
        match pool.get() {
            Ok(conn) => Outcome::Success(DbConn(conn)),
            Err(_) => Outcome::Failure((Status::ServiceUnavailable, ())),
        }
    }
}
```

### Adding the Request Handlers

Essentially the final part of this is to add in the request handlers. Thanks to
Rocket's code generation this is quite easy to write. For example to add the
request to return all the tasks the request would look like:

```rust
#[get("/")]
fn get_all_tasks(conn: DbConn) -> Json<Vec<Task>> {
    Json(Task::all(&conn))
}
```

Then we would update the `start` function to mount this route:

```rust
pub fn start() {
    rocket::ignite()
        .mount("/tasks", routes![get_all_tasks])
        .manage(init_pool())
        .launch();
}
```

#### Returning Some Data

However we have this new `Json` type that we are returning from our request so
at the moment this won't compile. Let's go ahead and add that type and make our
`Task` model serializable.

_Just as side note I wouldn't recommend in practice you have you database
representation the same the model type returned from your request but for the
sake of brevity for this walk through I will keep it this way_

To obtain the `Json` type we will add a dependency from Rocket,
`rocket-contrib`, adding to our `Cargo.toml`

```toml
[dependencies.rocket_contrib]
version = "0.3.6"
default-features = false
features = ["json"]
```

Then we need to make our `Task` serializable, and our `NewTask` whilst we are at
it. To do this we will use `serde` which is the standard serialization and
de-serialization crate used in Rust. With a small tweak of our `Cargo.toml` and
some added `derive`s to our type we are done.

`Cargo.toml`

```toml
serde = "1.0"
serde_json = "1.0"
serde_derive = "1.0"
```

`model.rs`

```rust
#[derive(Deserialize, Serialize, Queryable)]
pub struct Task {
    id: i32,
    task: String,
    completed: bool,
}

#[derive(Deserialize, Serialize, Insertable)]
#[table_name = "tasks"]
pub struct NewTask {
    task: String,
    completed: bool,
}
```

#### Adding the Rest of the Requests

Following the initial request we can add the remaining operations to create,
update, and delete tasks.

```rust
#[post("/", data = "<task>")]
fn create_task(task: Json<NewTask>, conn: DbConn) -> Json<String> {
    Task::create(&conn, task.into_inner());
    Json("Task added".to_owned())
}

#[put("/<id>", data = "<task>")]
fn update_task(id: u32, task: Json<Task>, conn: DbConn) -> Json<i32> {
    let id = Task::update(&conn, id as i32, task.into_inner());
    Json(id)
}

#[delete("/<id>")]
fn delete_task(id: u32, conn: DbConn) -> Json<Value> {
    Task::delete(&conn, id as i32);
    Json(json!({"status": "ok"}))
}
```

Then finally don't forget to mount these routes when you start `rocket` and we
have ignition!.

[rocket]: https://rocket.rs/
[rocket-tutorial]: https://rocket.rs/guide/state/#databases

## Gotham

[Gotham][gotham] is a web framework a bit newer on the scene. With one of the
key differences between it and Rocket is that it targets **only** stable Rust.
Whereas to get those really fancy macros with Rocket it is currently only able
to run on nightly Rust, which may be a deal breaker for some. This is not to say
this is the only differing aspect but a notable one.

Being a bit newer, Gotham is still changing it's shape and structure as when
writing this it was only **v0.2**. However they do provide numerous examples of
how to use the framework in the several ways in which it was designed. Due to
the youth of this framework some of the examples on the website are out of date
as the framework changes but the [GitHub][gotham-gh] does provide up to date
examples. This is what I used to piece this one together.

### Finding Common Ground

The way I approached this little project was to try the different web frameworks
whilst keeping the same database interactions. Therefore it seemed sensible to
extract the common database work and share that between the two frameworks.
Therefore we will start with the current `model.rs` and keep it as similar as
possible.

### Getting it started

As I was less familiar with this framework and the examples weren't yet as
documented I thought I would try some nice and simple routing and basic
responses for those endpoints but they didn't need to return anything.

The first part was simply starting the framework, this looked a bit different
but still pretty simple:

```rust
pub fn start() {
    let addr = "127.0.0.1:8000";
    println!("Listening for requests at http://{}", addr);
    gotham::start(addr, router())
}
```

You will note there is a `router()` function which I have made no mention of,
never fear I will explain it here! Rather than the dispersed approach for the
routing that was used in Rocket where each route gets to be defined at the
function that is to be executed, in Gotham we define the routes in a single
function producing a `Router`. Whilst it looks different we are essentially
doing the same thing as we had to mount all the routes in Rocket when starting
it.

```rust
use gotham::router::Router;
use gotham::router::builder::*;

fn router -> Router {
    build_simple_router(|route|{
        route.get("/tasks").to(get_all_tasks);
    })
}
```

So now we have a simple route to the `get_all_tasks` request handler so we
better define that on but first when we look into the documentation for Gotham
it has a very clear definition of what a `Handler` must look like. It takes a
`State` and returns a tuple of `(State, Response)`. Now we want to return a list
of `Task`s, so thankfully this is made very simple by using the `IntoResponse`
trait provided from Gotham. With a small tweak of our `model.rs` we can have our
response type now be a `TaskList`.

`model.rs`

```rust
// ...

#[derive(Deserialize, Serialize)]
pub struct TaskList {
    pub list: Vec<Task>,
}
```

Gotham uses `hyper` and `mime` crates to define its response structure so we
will need to grab those.

```rust
extern crate hyper;
extern crate mime;
extern crate serde_json;

use gotham::state::State;
use gotham::http::response::create_response;
use hyper::{Response, StatusCode};

impl IntoResponse for TaskList {
    fn into_response(self, state: &State) -> Response {
        create_response(
            &state,
            StatusCode::Ok,
            Some((
                serde_json::to_string(&self.list)
                    .expect("serialized product")
                    .into_bytes(),
                mime::APPLICATION_JSON,
            )),
        )
    }
}
```

Now we are ready to write our `Handler` to retrieve all the tasks.

```rust
fn get_all_tasks(state: State) -> (State, TaskList) {
    let tasks = vec![
        Task {
            id: 1,
            task: "Do homework".to_owned(),
            completed: false,
        },
    ];
    (state, tasks)
}
```

### Manage the Database Connection

Now we want to connect this up so it actually reads from the database. Here I
followed a similar approach to what I had seen from working with Rocket, which I
am sure is probably not the best or intended method with Gotham but it has
seemed to do the job.

_After putting this together I did actually see that Gotham does have some work
in their GitHub repository about getting Diesel to connect with it._

The way Gotham manages state is though its definition of a `Handler` as we have
already seen. Every `Handler` is given the state, what we need to do is tell
Gotham that we would like to add our database connection to part of that state
for it to manage. After some playing around I found the simplest way to do this
is through middleware.

In Gotham I have understood the concept of `middleware` as a construct to allow
the management of the requests. They are called before the request is sent to
the `Handler` and if desired can actually manage the response of the `Handler`.
The Gotham framework manages the calling of these we simple have to implement
the `Middleware` trait and add it to our `Router`.

First of all we want our database connection pool to be able to be managed. To
do so we add `gotham-derive` to our dependencies and create the following
struct:

```rust
#[derive(StateData)]
struct PoolState(Pool);
```

Now we are able to add `PoolState` to the Gotham `State`. Next we create our
middleware and implement it. The connection pool is created in the exact same
manner as it was when using Rocket.

```rust
use gotham::middleware::Middleware;

#[derive(Clone, NewMiddleware)]
struct DbConnMiddleware;

impl Middleware for DbConnMiddleware {
    fn call<Chain>(self, mut state: State, chain: Chain) -> Box<HandlerFuture>
    where
        Chain: FnOnce(State) -> Box<HandlerFuture>,
    {
        if !state.has::<PoolState>() {
            // Initialize it
            state.put(PoolState(init_pool()));
        }
        chain(state)
    }
}
```

Finally we modify our `Router` to use this middleware:

```rust
use gotham::pipeline::single::single_pipeline;

fn router() -> Router {
    let (chain, pipelines) = single_pipeline(new_pipeline().add(DbConnMiddleware).build());
    build_router(chain, pipelines, |route| {
        route.get_or_head("/").to(index);
        route.get("/tasks").to(get_all_tasks);
    })
}
```

### Adding the Read All Functionality

Now that we have access to the database through the `State` it is actually very
simple to implement the CRUD functionality. First we create a simple helper
method to extract the database connection from the state.

```rust
fn db_conn(state: &State) -> Option<DbConn> {
    state.borrow::<PoolState>().get().ok().map(|x| DbConn(x))
}
```

Then we can complete our read all functionality:

```rust
fn get_all_tasks(state: State) -> (State, TaskList) {
    let conn = db_conn(&state).expect("Failed with DB connection");
    let tasks = TaskList {
        list: Task::all(&conn),
    };
    (state, tasks)
}
```

### Using Path Variables

The finally piece of the puzzle is implementing the update and delete where we
specify the ID of the task we want to use. This is provided in the URL path.
This is very easy to implement and reuse for both of these by creating a struct
that Gotham will populate from the path.

```rust
#[derive(Deserialize, StateData, StaticResponseExtender)]
struct PathId {
    id: u32,
}
```

Followed by changing the `Router` definition to expect there to be a path
variable

```rust
// ... previous router definition
route
    .put("/task/:id")
    .with_path_extractor::<PathId>()
    .to(update_task);
route
    .delete("/task/:id")
    .with_path_extractor::<PathId>()
    .to(delete_task);
// ... rest of definition
```

Then we can define our delete to look like this:

```rust
fn delete_task(mut state: State) -> (State, Response) {
    let PathId { id } = PathId::take_from(&mut state);
    let conn = db_conn(&state).expect("Failed with DB connection");
    Task::delete(&conn, id as i32);
    let resp = create_response(&state, StatusCode::Ok, None);
    (state, resp)
}
```

### Using the Request Body

You may have noticed the update and create have yet to be addressed and this is
because these needed to extract the `Task` to create or update from the body of
the request. This was a little bit more clunky than simply adding annotations
but once I got around it, it wasn't too bad at all.

Firstly you must know that Gotham actually stores the `Body` as part of the
`State` provided to the handler so that is simply how we would extract it.

```rust
use hyper::Body;

let body = Body::take_from(&mut state)
        .concat2()
        .then(// Do something with the body);
```

Now there is a fair bit of boiler plate to get this `Body` out of the state as
the action is asynchronous. So I thought it would be nice if I could just
provide a closure of what I want to do with the body once it is ready and not
have to write this boiler plate out each time (I know twice in this case but I
was determined). So after some fighting with the borrow checker with lifetimes
this was the resulting create and update functions I got:

```rust
fn body_handler<F>(mut state: State, f: F) -> Box<HandlerFuture>
where
    F: 'static + Fn(String, &State) -> Response,
{
    let body = Body::take_from(&mut state)
        .concat2()
        .then(move |full_body| match full_body {
            Ok(valid_body) => {
                let body_content = String::from_utf8(valid_body.to_vec()).unwrap();
                let res = f(body_content, &mut state);
                future::ok((state, res))
            }
            Err(e) => return future::err((state, e.into_handler_error())),
        });
    Box::new(body)
}

fn create_task(state: State) -> Box<HandlerFuture> {
    body_handler(state, |s, state| {
        let task = serde_json::from_str(&s).expect("Failed to deserialize");
        let conn = db_conn(state).expect("Failed with DB connection");
        Task::create(&conn, task);
        create_response(state, StatusCode::Ok, None)
    })
}

fn update_task(mut state: State) -> Box<HandlerFuture> {
    let PathId { id } = PathId::take_from(&mut state);
    body_handler(state, move |s, state| {
        let task = serde_json::from_str(&s).expect("Failed to deserialize");
        let conn = db_conn(&state).expect("Failed with DB connection");
        Task::update(&conn, id as i32, task);
        create_response(state, StatusCode::Ok, None)
    })
}
```

[gotham]: https://gotham.rs/
[gotham-gh]: https://github.com/gotham-rs/gotham/tree/master/examples

## Conclusion

All in all I hope I could give some really basic use cases for working with
these frameworks and I hope to keep adding to this and improving the code as I
go so keep an eye on the [repository][repo] if you are interested. 

The key thing I wanted to try with this exercise was usability of the
frameworks, not really comparing in terms of performance. I think in this small
example Diesel was extremely easy to work with, mainly due to its helpful
documentation and examples.

In terms of the web frameworks you could definitely feel that Rocket was the
more mature in particular with its documentation which really assisted in
getting everything started. Further having those macros made everything very
simple to get it together, so if speed to get a product is your thing I would
definitely recommend. Also the variety of functionality Rocket currently
supports is great, I have barely scratched the surface with this basic example.

Gotham definitely has a lot going for it and it has very clear goals of what it
wants to achieve. It does have a bit more boiler plate lying around but
personally I don't mind that because I feel like I can understand a bit better
how things are working. This is definitely a promising framework and look
forward to seeing what is to come.

[repo]: https://github.com/maccoda/micro-rs
