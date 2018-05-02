+++
date = 2018-05-01
title = "WIP - Making microservices in Rust"
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

## Frameworks

So what were the frameworks that I wanted to explore?

### [Diesel](https://diesel.rs/)

Diesel is one of the more popular [ORM][orm] in the Rust community. It has some
great documentation to get set up and to understand how to use it. It also comes
with a very handy CLI tool to assist in the setting up of a project and managing
database migrations, which is great for keeping everything nice and uniform and
automated. A great combination indeed!

The [getting started guide](http://diesel.rs/guides/getting-started/) for Diesel
provides a solid foundation on how to put it all together so I won't try rewrite
that, rather provide what I needed to get these tasks together.

#### Setting up the migrations

Following the guide the CLI tool will create an `up.sql` and `down.sql` in the
`migrations` directory. In here we want to put the SQL query to create our table
and the opposing query to undo the change.

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


[orm]:https://en.wikipedia.org/wiki/Object-relational_mapping
[repo]:https://github.com/maccoda/micro-rs