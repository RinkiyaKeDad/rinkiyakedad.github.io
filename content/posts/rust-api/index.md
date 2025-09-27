---
title: "Creating a REST API in Rust"
read_time: true
date: 2025-09-27T00:00:00+05:30
tags:
  - rust
---

I recently started learning Rust. After watching a bunch of [bigboxSWE videos](https://www.youtube.com/watch?v=QMbx0dTWJIQ), I was influenced by his idea of “learning by doing.” I decided to keep things simple and start by building a REST API that supports CRUD (Create, Read, Update, Delete) functionality. Turns out, when it comes to Rust, even this wasn’t as simple as I thought. But I learned A LOT in the process. In this blog, I’ll walk you through the steps of creating a simple API in Rust and share all the things I learned along the way. Hopefully, this helps you in your journey of learning Rust :)

# What we’ll be building

We’ll be building a REST API that supports CRUD operations. CRUD operations, for those of you who aren’t familiar, mean you can send requests to **create** new entries, **read** a particular entry or all entries, **update** an entry, or **delete** an entry. Our API will be storing these entries in a PostgreSQL database, which we’ll be running locally using Docker. And what will these entries actually be about?

Well, I’m a huge fan of board games, so that’s what I’m going to go with for this tutorial. We’ll be creating an API that stores the name of a board game, who created it, and how many times it has been played by the user. Now that we have an idea of what we’re building, let’s get started.

# Creating a server using Axum

I’ll assume you have Rust and Cargo installed. If not, you can find the instructions for that [here](https://doc.rust-lang.org/cargo/getting-started/installation.html). We’ll run the following commands to initialize our project:

```bash
cargo run rust-crud-api
cd rust-crud-api
```

Among the files that get created, you’ll notice a `Cargo.toml` file. This is where all the dependencies for our Rust project are stored. Update the dependencies section with this so that we have everything we need to build our API.

```toml
# Cargo.toml

[dependencies]
axum = "0.8.4"
serde_json = "1.0.143"
tokio = { version = "1", features = ["full"] }
sqlx = { version = "0.8", features = [ "runtime-tokio", "tls-rustls-ring-webpki", "postgres", "chrono", "uuid" ] }
dotenv = "0.15.0"
serde = { version = "1.0.219", features = ["derive"] }
chrono = { version = "0.4.41", features = ["serde"] }
uuid = { version = "1.17.0", features = ["serde", "v4"] }
```

Now we’re ready to build a simple server that returns “Hello, World!” as a response when we send it a request. Replace the code in `main.rs` with this, and then I’ll walk you through what’s happening.

```rust
use axum::{Json, Router, response::IntoResponse, routing::get};
use serde_json::json;

#[tokio::main]
async fn main() {
    // build our application with a single route
    let app = Router::new().route("/api", get(hello_world));

    // listen globally on port 3000
    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    println!("Server started successfully at 0.0.0.0:3000");
    axum::serve(listener, app).await.unwrap();
}

async fn hello_world() -> impl IntoResponse {
    let json_response = json!({
        "status": "ok",
        "message": "Hello, World!"
    });
    Json(json_response)
}
```

We’re using `axum` to build our REST API. [Axum](https://github.com/tokio-rs/axum) is one of the most popular choices when it comes to frameworks for building APIs in Rust. Next, we’re also using the crate `serde_json`. This provides a way to automatically map JSON data into Rust data structures.

After importing all the things we need, two things will stand out if this is your first time building an API in Rust:

1. We have a macro above our main function.
2. Our main function is `async fn main` instead of the usual `fn main`.

Let’s understand them. By default, Rust doesn't know how to run [asynchronous code](https://www.reddit.com/r/learnprogramming/comments/jtfq24/comment/gc59ujh/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button). You need a runtime to execute it. [Tokio](https://tokio.rs/) is that runtime, and `#[tokio::main]` is a macro that transforms the `async fn main()` into a synchronous `fn main()` which initializes a runtime instance and executes the async main function. So this:

```rust
#[tokio::main]
async fn main() {
    println!("hello");
}
```

would get transformed into this:

```rust
fn main() {
    let mut rt = tokio::runtime::Runtime::new().unwrap();
    rt.block_on(async {
        println!("hello");
    })
}
```

We need our server to be asynchronous so that we can efficiently process multiple requests. If we don’t make our functions asynchronous, then one request would block the thread until it’s finished, and no other requests would be able to get processed.

After that, we use the `Router` struct from Axum to create a new Router and then call the `route` method on that Router instance. 

```rust
let app = Router::new().route("/api", get(hello_world));
```

To this `route` method, we pass the URL where we want this to be accessible, and the `get` function, which creates `GET` route handlers. What we pass to `get` is called a “handler,” and that is the code that gets executed whenever a `GET` request reaches our `/api` endpoint. After initializing our Router, we have some standard code to bind the port for our server and then start it.

Let’s now look at our handler code, which lives in the `hello_world` function. It’s important to understand the difference between the `json!` macro from `serde_json` and the `Json` wrapper type from Axum here. `json!` creates a Rust data structure (a `serde_json::Value`) and not JSON bytes. So this:

```rust
let json_response = json!({
    "status": "ok",
    "message": "Hello, World!"
});
```

gets converted to:

```rust
// This is what json! creates - Rust data in memory:
Value::Object({
    "status": Value::String("ok"),
    "message": Value::String("Hello, World!")
})
```

`Json`, on the other hand, takes that Rust data and converts it into an HTTP response that can be sent over the network. It also sets the `Content-Type: application/json` header. We need to use `json!` because `Json` expects serializable data, not a raw string. So we **can’t** do something like this:

```rust
Json("{
    \"status\": \"ok\",
    \"message\": \"Hello, World!\"
}")
```

“Serializable data” is data that can be converted to a standard format (JSON, XML, binary, etc.). If you just pass `Json` the raw string like above, it’s not in a serializable format, and `Json` wouldn’t know what to do with it.

# **Testing our “Hello, World!” server**

Alright, now that we’ve understood different parts of our code, it’s time to test it out. Run the code using `cargo run` and send the following curl request.

```bash
curl -X GET [http://localhost:3000/api](http://localhost:3000/api) -s | jq .
```

I’m using [jq](https://github.com/jqlang/jq) so that we see the result in a pretty way, but you don’t have to use that. At this point, you should see a response saying “Hello, World!” from the server. Congrats, we just built our first server in Rust! :)

Now let’s do a bit of restructuring for our project. Let’s move our `hello_world` handler to a new file called `handlers.rs`. This will help organize things better when we add more handlers. Since it’s in a different file (each file is a separate module in Rust), we need to add `pub` before the function name to make it public, and then import it in `main.rs` using `mod handlers`.

```rust
// main.rs
use axum::{Router, routing::get};

mod handlers;
use handlers::hello_world;

#[tokio::main]
async fn main() {
    // build our application with a single route
    let app = Router::new().route("/api", get(hello_world));

    // run our app with hyper, listening globally on port 3000
    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    println!("Server started successfully at 0.0.0.0:3000");
    axum::serve(listener, app).await.unwrap();
}

// handler.rs
use axum::{Json, response::IntoResponse};
use serde_json::json;

pub async fn hello_world() -> impl IntoResponse {
    let json_response = json!({
        "status": "ok",
        "message": "Hello, World!"
    });
    Json(json_response)
}
```

# Setting up our PostgreSQL database

We’ll be using PostgreSQL (also called Postgres) as the database to store all the information related to our board games. We’ll use [Docker](https://docs.docker.com/engine/install/) and Docker Compose to run this locally, so make sure you have it installed. Create the following Docker Compose file to run Postgres locally:

```yaml
services:
  postgres:
    image: postgres:latest
    container_name: postgres
    env_file:
      - ./.env
    ports:
      - '5432:5432'
    volumes:
      - postgresDb:/var/lib/postgres
volumes:
  postgresDb:
```

If you’re not familiar with Docker or Docker Compose, I have a [series of blogs](https://dev.to/rinkiyakedad/introduction-to-docker-1hp2) covering the basics in detail that you can check out. But a quick summary of what’s happening here is that we use the official `postgres` container image to run it locally on port `5432` (the default port for PostgreSQL). We also pass it some environment variables from the `.env` file (which we’ll create later) to provide configuration. Finally, we use a named volume in Docker to persist data. Without volumes, when we stop or restart the container, we’d lose every record we saved in it. We mount to `/var/lib/postgres` because that’s where PostgreSQL stores its data inside the Docker container. When you use a named volume like `postgresDb`, Docker stores the data in its internal storage area, not in our project directory, so we don’t need to specify a path for where to store it on our local machine. Docker manages the location for us.

Next, let’s create a `.env` file with the configuration for our database.

```
POSTGRES_DB=gamestats
POSTGRES_USER=admin
POSTGRES_PASSWORD=password

DATABASE_URL=postgres://${POSTGRES_USER}:${POSTGRES_PASSWORD}@localhost:5432/${POSTGRES_DB}
```

With these two things configured, we can start our database using the `docker compose up -d` command. To stop the container, you can run `docker compose stop`. If you want to remove the container and the volume associated with it (that is, remove the data stored in the DB as well), you can run `docker compose down -v`. Now let’s add some code to connect to this running Postgres database.

# Connecting to the PostgreSQL Database

Update your `main.rs` file to look like this:

```rust
// omitted imports

pub struct AppState {
    db: PgPool,
}

#[tokio::main]
async fn main() {
    dotenv().ok();
    let db_url = std::env::var("DATABASE_URL").expect("DATABASE_URL must be set");
    let pool = match PgPoolOptions::new()
        .max_connections(10)
        .connect(&db_url)
        .await
    {
        Ok(pool) => {
            println!("Connected to DB successfully");
            pool
        }
        Err(err) => {
            println!("Failed to connect to DB: {}", err);
            std::process::exit(1);
        }
    };

    let app = Router::new()
        .route("/api", get(hello_world))
        .with_state(Arc::new(AppState { db: pool.clone() }));

    // run our app with hyper, listening globally on port 3000
    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    println!("Server started successfully at 0.0.0.0:3000");
    axum::serve(listener, app).await.unwrap();
}
```

Let’s walk through the changes one by one. The first thing we do is create a new `struct` called `AppState`. This will be used to share the database connection across different parts of our application. Instead of passing the database connection to every function, we’ll pass the entire `AppState` struct.

Next, we use `dotenv`. Without `dotenv`, our app would only see environment variables set in our shell, not the ones in our `.env` file. `dotenv().ok()` essentially means: "Try to load the `.env` file, but I don't care if it fails, just ignore the result." That’s what the `.ok()` part does. It converts the [`Result`](https://doc.rust-lang.org/std/result/) type into an [`Option`](https://doc.rust-lang.org/std/option/) type.

Next, we read the database URL from the environment variables we set and create a connection pool. We can think of a database pool like a fleet of taxis for our database connections. Instead of calling a new taxi (connection) every time we need to go somewhere (make a query), we have a fleet ready to pick us up immediately. When we’re done, the taxi returns to the fleet for the next person to use. This makes our web server much faster and more efficient at handling database operations. In the configuration options of this pool, we set `max_connections` to 10. This limits how many simultaneous connections our Rust application can have open to the PostgreSQL database at any given time.

```rust
// Let's suppose our app can handle 1000 HTTP requests per second
// But it can only use 10 database connections maximum

// Request 1: Uses connection #1
// Request 2: Uses connection #2  
// Request 3: Uses connection #3
// ...
// Request 10: Uses connection #10
// Request 11: Waits for a connection to become available
// Request 12: Waits for a connection to become available
```

After creating our DB pool, the other change you’ll notice is that we pass a state to our router using `Arc`. `Arc` stands for "Atomically Reference Counted" and is a way to share data between multiple parts of our program safely. Let’s understand this in a bit more detail since it’s related to an important Rust concept: ownership.

In Rust, you can only have one owner of data at a time. So this won’t work:

```rust
let data = String::from("Hello");

// This moves ownership - `data` is no longer available here
let moved_data = data;

// This won't work! `data` was moved
println!("{}", data); // Error: value used after move
```

Which means if multiple handlers in our web server need access to the database pool, we can’t do something like this:

```rust
// This won't work - can't move the same data to multiple places
let pool1 = pool;        // Moves pool
let pool2 = pool;        // Error: pool already moved
```

This is also the reason that when creating an object from the `AppState` struct, we do `db: pool.clone()` and not `db: pool`. If we did the latter, then `pool` would no longer be available to use in the rest of the code. We also can’t pass `&pool` because references have lifetime requirements. If the Rust compiler can't guarantee that `pool` will live long enough, it will not accept us passing a reference.

Using Arc avoids the ownership problem by letting us have multiple owners of the same data.

```rust
use std::sync::Arc;

fn main() {
    let data = String::from("Hello");
    let shared_data = Arc::new(data); // Wrap in Arc

    // Now we can have multiple references
    let reference1 = Arc::clone(&shared_data); // Doesn't copy the data!
    let reference2 = Arc::clone(&shared_data); // Just increments a counter

    println!("{}", shared_data); // Works
    println!("{}", reference1); // Works  
    println!("{}", reference2); // Works
}
```

Arc implements a counter that keeps track of references to the actual data instead of copying the data each time (which is expensive).

```rust
┌─────────────────┐
│   Arc<String>   │
│                 │
│  ┌─────────────┐│
│  │ "Hello"     ││ ← The actual data
│  └─────────────┘│
│                 │
│  Reference      │
│  Count: 3       │ ← How many owners exist
└─────────────────┘
    ↑     ↑     ↑
    │     │     │
  ref1  ref2  ref3
```

When the last reference to the data is dropped, Arc automatically cleans up the data. Ownership can be tricky to understand in Rust at first, and I hope you’re getting somewhat of an idea. You don’t need to fully understand how Arc works to follow along, but I wanted to touch upon why we’re using it. If you’d like to learn more about Arc, I’d recommend checking [this](https://itsallaboutthebit.com/arc-mutex/) out.

When we run our code now, we should see the following output:

```bash
$ cargo run
warning: field `db` is never read
  --> src/main.rs:11:5
   |
10 | pub struct AppState {
   |            -------- field in this struct
11 |     db: PgPool,
   |     ^^
   |
   = note: `#[warn(dead_code)]` on by default

warning: `rust-crud-api` (bin "rust-crud-api") generated 1 warning
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.30s
     Running `target/debug/rust-crud-api`

Connected to DB successfully
Server started successfully at 0.0.0.0:3000
```

We’ll get rid of the warning soon enough, but so far we’ve successfully managed to connect to the DB, and that’s a win!

# Running our database migrations

Running a database migration means making a change to the schema of our database. Since we don’t have a schema yet for our DB, the first migration we need is the one that creates our database and its schema. This would be our “up” migration. And the “down” migration we’ll use will be the one that lets us go back to our previous state, that is, dropping the table. To do this, we’ll use the [SQLx CLI](https://crates.io/crates/sqlx-cli), which you can install by running:

```bash
cargo install sqlx-cli
```

Then create both the up and down migration files by running:

```bash
sqlx migrate add -r create_games_table
```

Edit the `some_random_number_create_games_table.up.sql` file to have the following contents:

```sql
-- Add up migration script here
CREATE TABLE IF NOT EXISTS games (
    id UUID PRIMARY KEY NOT NULL,
    name TEXT NOT NULL UNIQUE,
    creator TEXT NOT NULL,
    plays INTEGER NOT NULL,
    created_at TIMESTAMPTZ NULL DEFAULT NOW()
);
```

I won’t go into the details of this SQL syntax, but what it essentially says is that our games should have a primary key of `id`, a `name` which is unique, a `creator`, number of `plays`, and a `created_at` timestamp that will be populated automatically. For our down migration file, we simply remove this table if it exists:

```sql
-- Add down migration script here
DROP TABLE IF EXISTS games;
```

Let’s run our “up” migration script by running:

```bash
sqlx migrate run
```

Now we can run the following commands to check if our database table was created with the right schema or not (make sure the Postgres container is running):

```bash
$ docker exec -it postgres bash
$ root@6c22115fva06: psql -U admin -d gamestats
$ gamestats= \dt
             List of relations
 Schema |       Name       | Type  | Owner
--------+------------------+-------+-------
 public | _sqlx_migrations | table | admin
 public | games            | table | admin
(2 rows)
$ gamestats= \d games
                          Table "public.games"
   Column   |           Type           | Collation | Nullable | Default
------------+--------------------------+-----------+----------+---------
 id         | uuid                     |           | not null |
 name       | text                     |           | not null |
 creator    | text                     |           | not null |
 plays      | integer                  |           | not null |
 created_at | timestamp with time zone |           |          | now()
Indexes:
    "games_pkey" PRIMARY KEY, btree (id)
    "games_name_key" UNIQUE CONSTRAINT, btree (name)
```

Once you’ve verified this, you can type “exit” twice to return to your local terminal. `sqlx migrate revert` will run the “down” migration script and get rid of this database table (and any data you might have saved in it so far).

# Creating the database model

To define our database model, let’s create a new file called `src/model.rs` with the following code:

```rust
// model.rs

use serde::{Deserialize, Serialize};
use uuid::Uuid;

#[derive(Debug, Deserialize, Serialize, sqlx::FromRow)]
pub struct GameModel {
    pub id: Uuid,
    pub name: String,
    pub creator: String,
    pub plays: i32,
    pub created_at: Option<chrono::DateTime<chrono::Utc>>,
}

```

The `#[derive(...)]` syntax tells Rust to automatically generate implementations of certain traits for our struct. In our case, the four traits being derived are:

1. Debug: Allows us to print the struct for debugging purposes. So we can use `println!("{:?}", game)` to see the struct's contents.
2. Deserialize: Allows converting JSON (or other formats) into our struct. This is needed when receiving data from HTTP requests to convert that into our struct.
3. Serialize: Allows converting our struct into JSON (or other formats). Used to send data in HTTP responses.
4. `sqlx::FromRow`: Allows converting database rows into our struct. This will be used with the `sqlx` library for database operations (we’ll see this soon).

Without this derive macro, we’d have to manually write a lot of lines of code to get all the above-mentioned functionality.

The other thing I wanted to explain was `created_at: Option<chrono::DateTime<chrono::Utc>>`.

`Option` is an enum that represents the possibility of a value being present or absent. The `Option` enum has two variants:

- `Some(T)`: Indicates that a value of type `T` is present.
- `None`: Indicates the absence of a value. This is conceptually similar to `null` in other programming languages.

We need to use `Option` here because, as per our database schema, the `created_at` field is optional, so it might not always have a value (though it very likely will since it’ll be automatically populated by Postgres when it creates an item in the table). After that, we use the chrono crate for handling dates and times.

# Creating the API Schema

Create the `src/schema.rs` file, which is where we’ll be writing the code for our API Schema. The API schema will be a struct that defines what our API request should look like. Add the following code to the file:

```rust
// schema.rs 

use serde::{Deserialize, Serialize};

/// Schema for creating or updating a player
#[derive(Serialize, Deserialize, Debug)]
pub struct GameSchema {
    pub name: String,
    pub creator: String,
    pub plays: i32,
}
```

Now, when trying to create a new game, we can send a request like this:

```json
{
  "name": "Catan",
  "creator": "Klaus Teuber",
  "plays": 0
}
```

And this is what will get stored in the database:

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",  // Auto-generated
  "name": "Catan",
  "creator": "Klaus Teuber", 
  "plays": 0,
  "created_at": "2025-09-19T10:30:00Z"  // Auto-generated
}
```

# Creating the handler for POST requests

The first handler we’ll create is the one that allows us to store a game in the database by sending a `POST` request. This is where you’ll start to see all the code we wrote for the schema and model start to come together. Update the code in your `handler.rs` and then we’ll go through it line by line to understand what’s happening:

```rust
// handler.rs

use std::sync::Arc;

use axum::{Json, extract::State, http::StatusCode, response::IntoResponse};
use serde_json::json;

use crate::{AppState, model::GameModel, schema::GameSchema};

pub async fn create_game_handler(
    State(data): State<Arc<AppState>>,
    Json(body): Json<GameSchema>,
) -> Result<impl IntoResponse, (StatusCode, Json<serde_json::Value>)> {
    let id = uuid::Uuid::new_v4();
    let game = sqlx::query_as!(
        GameModel,
        r#"INSERT INTO games (id, name, creator, plays) VALUES ($1, $2, $3, $4) RETURNING *"#,
        &id,
        &body.name,
        &body.creator,
        &body.plays
    )
    .fetch_one(&data.db)
    .await
    .map_err(|e| e.to_string());

    if let Err(err) = game {
        if err.to_string().contains("duplicate key value") {
            let error_response = serde_json::json!({
                "status": "error",
                "message": "Game already exists",
            });
            return Err((StatusCode::CONFLICT, Json(error_response)));
        }

        return Err((
            StatusCode::INTERNAL_SERVER_ERROR,
            Json(json!({"status": "error","message": format!("{:?}", err)})),
        ));
    }

    let game_response = json!({
            "status": "success",
            "data": json!({
                "game": game
        })
    });

    Ok(Json(game_response))
}
```

First, let’s understand the function parameters for our `create_game_handler` function:

- `State(data): State<Arc<AppState>>`: extracts the database connection from Axum. This will be used to connect to the DB when saving a record there.
- `Json(body): Json<GameSchema>`: extracts JSON data from the HTTP request. This includes the details of the game we’re trying to save.

You might be wondering why we can’t do this:

```rust
// This WON'T work
pub async fn create_game_handler(
    data: Arc<AppState>,        
    body: GameSchema,
) -> Result<...>
```

This is because when we send an HTTP request, it’s not just “here’s some data.” It’s a complex object with:

- Headers (Content-Type, Authorization, etc.)
- Body (the actual JSON data)
- Path parameters (like `/users/123`)
- State (shared application data)
- Query parameters (like `?page=1&limit=10`)

The problem then is: how does Rust know that `data` should come from the application state, or that `body` should come from the JSON request body? The syntax `State(data): State<Arc<AppState>>` is telling Axum:

1. To extract the application state
2. Make sure it’s of type `Arc<AppState>` 
3. Put the extracted value in a variable called `data` 

Axum has many extractors for different types of data:

```rust
pub async fn complex_handler(
    State(data): State<Arc<AppState>>,           // Application state
    Json(body): Json<GameSchema>,                // JSON from request body
    Query(params): Query<HashMap<String, String>>, // Query parameters
    Path(id): Path<Uuid>,                        // Path parameter
    headers: HeaderMap,                          // All headers
) -> Result<...>
```

We’ll see some of these later when we create other endpoints for our API. So far, for the `create_game_handler`, only the application state and the JSON body extractors are relevant for us.

Now, let’s understand the return type for our handler function, which is `Result<impl IntoResponse, (StatusCode, Json<serde_json::Value>)>`.

`Result` is a type in Rust that is actually an enum with two variants: `Ok(T)` for a successful operation containing a value of type `T`, and `Err(E)` for a failed operation containing an error value of type `E`. It will return `Ok` with the value `T` if everything works correctly, or `Err` with the error `E` if not.

Our success type `T` in this case is `impl IntoResponse`, which basically means it can be any type that *implements* the IntoResponse [trait](https://doc.rust-lang.org/rust-by-example/trait.html). The IntoResponse trait says that the value should be able to be converted into an HTTP response. If this is getting confusing, think of our success type as saying: “I’ll return something that can become an HTTP response, but I won’t tell you exactly what type.”

For our error type `E` we have `(StatusCode, Json<serde_json::Value>)`, which is a tuple (pair) of values. The first part of it needs to be an HTTP status code (like 404, 500, etc.), and the second part is our actual error message, but it needs to be in JSON format.

I know that might have felt like a lot, considering we only just covered the function signature, but I really wanted you to understand these things here because we’ll be seeing this pattern again for our other handlers too.

After this, the first thing we do in our function body is use the `uuid` crate to create a random and unique ID for the particular game the user is trying to store. Then we have our main code, which is responsible for storing the game details in the database.

```rust
let game = sqlx::query_as!(
        GameModel,
        r#"INSERT INTO games (id, name, creator) VALUES ($1, $2, $3) RETURNING *"#,
        &id,
        &body.name,
        &body.creator
    )
    .fetch_one(&data.db)
    .await
    .map_err(|e| e.to_string());
```

`query_as` is a macro (evident by the `!`) from the `sqlx` crate. `sqlx` was also the CLI we installed earlier to run our DB migrations. It’s also our project dependency (see Cargo.toml), so we can use it in our code using the `sqlx::` syntax.

This macro creates a type-safe database query. It takes a SQL query (in the form of a raw string literal) and converts it into Rust code. It also ensures the query is valid at compile time. The `RETURNING *` at the end of our query tells Postgres to return the complete record that it inserted. This is where type safety comes into the picture and checks that the returned record matches the `GameModel` struct, which we specify as the first argument to our `query_as` macro.

After that, `.fetch_one()` basically asks `sqlx` to execute the query and return exactly one row. We’ll also take a look at `execute()` later, which tells `sqlx` to just execute the query and not return any data. We pass `fetch_one` the database connection (`&data.db`), which is borrowed from AppState and needed for it to talk to the DB.

And then we use `.await`, which pauses this function until the database responds. This is the asynchronous part that allows other code to run while waiting for the database. When the database responds, this function continues.

In `.map_err`, we use the anonymous function syntax in Rust. It converts our error message to a simple string for easier handling. This becomes relevant in the code below:

```rust
    if let Err(err) = game {
        if err.contains("duplicate key value") {
            let error_response = serde_json::json!({
                "status": "error",
                "message": "Game already exists",
            });
            return Err((StatusCode::CONFLICT, Json(error_response)));
        }

        return Err((
            StatusCode::INTERNAL_SERVER_ERROR,
            Json(json!({"status": "error","message": format!("{:?}", err)})),
        ));
    }
```

Here, if there was an error (which was converted to a string by our `map_err` function), we move it into an `err` variable and then check if it contains the phrase “duplicate key value.” If it does, we return the appropriate error response with the message. If not, then we return whatever is actually in the error message. We had learned about the usage of `Json` vs `json!` at the start, so I won’t be explaining that part of the code again.

Finally, if there was no error, we return our saved game object for the user to see.

Now let’s register this handler so that we can try it out. Create the following file `src/route.rs`.

```rust
// route.rs

use std::sync::Arc;

use axum::{Router, routing::post};

use crate::{AppState, handlers::create_game_handler};

pub fn create_router(app_state: Arc<AppState>) -> Router {
    Router::new()
        .route("/api/game", post(create_game_handler))
        .with_state(app_state)
}
```

Not a lot to explain here, except that we created a new router function, which will help us consolidate all the routes our API has. This is a much cleaner approach compared to having everything in the `main.rs` file. Finally, let’s update our `main.rs` to use this router.

```rust
// main.rs

use std::sync::Arc;

use dotenv::dotenv;
use sqlx::postgres::{PgPool, PgPoolOptions};

use crate::route::create_router;

mod handlers;
mod model;
mod route;
mod schema;

pub struct AppState {
    db: PgPool,
}

#[tokio::main]
async fn main() {
    dotenv().ok();
    let db_url = std::env::var("DATABASE_URL").expect("DATABASE_URL must be set");
    let pool = match PgPoolOptions::new()
        .max_connections(10)
        .connect(&db_url)
        .await
    {
        Ok(pool) => {
            println!("Connected to DB successfully");
            pool
        }
        Err(err) => {
            println!("Failed to connect to DB: {}", err);
            std::process::exit(1);
        }
    };

    let app = create_router(Arc::new(AppState { db: pool.clone() }));

    // listen globally on port 3000
    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    println!("Server started successfully at 0.0.0.0:3000");
    axum::serve(listener, app).await.unwrap();
}

```

Now, with this, we have everything we need for our create game endpoint. Run your code using `cargo run` and send the following curl request.

```bash
$ curl -X POST -H "Content-Type: application/json" -d '{"name": "Wingspan", "creator": "Elizabeth Hargrave", "plays": 23}' http://localhost:3000/api/games
 {
  "data": {
    "game": {
      "Ok": {
        "created_at": "2025-09-20T16:18:49.102219Z",
        "creator": "Elizabeth Hargrave",
        "id": "d1972ee2-36d0-4794-a8fe-2772c1956707",
        "name": "Wingspan",
        "plays": 23
      }
    }
  },
  "status": "success"
}
```

You should see a response similar to what I shared above, which is the completed game entry that was saved in the database. You’ll notice how the `created_at` and `id` fields were added automatically because of the code we wrote. If you try to send the same request again, you’ll notice an error saying that the game already exists in the database. This verifies that our game was actually saved. Let’s now create an endpoint for fetching all games and a single game by its ID.

# Creating the handler for GET requests

Add the following code to the `handlers.rs` file:

```rust
// handlers.rs

// other code

pub async fn game_list_handler(
    State(data): State<Arc<AppState>>,
) -> Result<impl IntoResponse, (StatusCode, Json<serde_json::Value>)> {
    // Query with macro
    let games = sqlx::query_as!(GameModel, r#"SELECT * FROM games ORDER by name"#)
        .fetch_all(&data.db)
        .await
        .map_err(|e| {
            let error_response = serde_json::json!({
                "status": "error",
                "message": format!("Database error: { }", e),
            });
            (StatusCode::INTERNAL_SERVER_ERROR, Json(error_response))
        })?;

    let json_response = serde_json::json!({
        "status": "ok",
        "count": games.len(),
        "notes": games
    });

    Ok(Json(json_response))
}
```

A lot of the concepts in this code are the same as what we discussed for the `create_game_handler` above, so this should be pretty easy to understand. The only thing different is the actual SQL query that we’re executing.

Next, add the handler that lets us fetch the details of a particular game (by specifying its ID) instead of all the games.

```rust
// handlers.rs

// other code

pub async fn get_game_handler(
    Path(game_id): Path<Uuid>,
    State(data): State<Arc<AppState>>,
) -> Result<impl IntoResponse, (StatusCode, Json<serde_json::Value>)> {
    let query_result = sqlx::query_as!(GameModel, r#"SELECT * FROM games WHERE id = $1"#, &game_id)
        .fetch_one(&data.db)
        .await;

    match query_result {
        Ok(game) => {
            let game_response = serde_json::json!({
                "status" : "success",
                "data": serde_json::json!({
                    "game": game
                })
            });

            Ok(Json(game_response))
        }
        Err(sqlx::Error::RowNotFound) => {
            let error_response = serde_json::json!({
                "status": "fail",
                "message": format!("Game with ID: {} not found", game_id)
            });
            Err((StatusCode::NOT_FOUND, Json(error_response)))
        }
        Err(e) => Err((
            StatusCode::INTERNAL_SERVER_ERROR,
            Json(json!({"status": "error", "message": format!("{:?}", e)})),
        )),
    }
}
```

Here in the function signature, we’re introduced to a new extractor from Axum, which I mentioned earlier, and that’s `Path`. If someone sends a request to `/api/games/fe323-ecfd2`, `Path` will extract the URL parameter `fe323-ecfd2` for us. We pass this extracted parameter to our SQL query to fetch the record whose game ID (unique for each game) matches what the user entered in the parameter. Other than that, the rest of the code is similar to what we’ve seen before for the other handlers.

# Creating the handler for DELETE requests

Now we’ll add the code that lets us delete a game from the database when we send a request with the game ID as a URL parameter. We’ll return this deleted game along with a success message in case we were successful. Append the following code to your `src/handlers.rs` file:

```rust
// src/handlers.rs

// other code

pub async fn delete_game_handler(
    Path(game_id): Path<Uuid>,
    State(data): State<Arc<AppState>>,
) -> Result<impl IntoResponse, (StatusCode, Json<serde_json::Value>)> {
    let query_result = sqlx::query_as!(
        GameModel,
        r#"DELETE FROM games WHERE id = $1 RETURNING *"#,
        &game_id
    )
    .fetch_one(&data.db)
    .await
    .map_err(|e| match e {
        sqlx::Error::RowNotFound => (
            StatusCode::NOT_FOUND,
            Json(json!({
                "status": "error",
                "message": "Game not found"
            })),
        ),
        _ => (
            StatusCode::INTERNAL_SERVER_ERROR,
            Json(json!({
                "status": "error",
                "message": format!("{:?}", e)
            })),
        ),
    })?;

    let response = json!({
        "status": "success",
        "message": "Game delete successfully",
        "data": {
            "deleted_game" : query_result
        }
    });

    Ok(Json(response))
}

```

The only thing that is a little different here is that we use a [match](https://doc.rust-lang.org/rust-by-example/flow_control/match.html) block in our `map_err`. `match` in Rust is like `switch` in C. In this `match` block, we first check if the error is `sqlx::Error::RowNotFound`. If it is, then we send a message telling the user that the game they’re trying to delete doesn’t exist. Otherwise, for all other error messages, we simply output whatever the message is for the user to see.

# Creating the handler for PATCH requests

PATCH requests allow us to modify parts of an already saved game in the database. Let’s suppose you want to update the number of plays of a game. Instead of deleting the original record and creating a new one, you can send a patch request which updates the `plays` field with the new count. But since any of the fields could be updated (other than timestamp and game ID), we need a new schema instead of our original schema, because, as per the `GameSchema` setting all fields when sending the request is mandatory, which might not be true in this case. Update the [`schema.rs`](http://schema.rs) file to look like this:

```rust
// schema.rs

use serde::{Deserialize, Serialize};

/// Schema for creating or updating a player
#[derive(Serialize, Deserialize, Debug)]
pub struct GameSchema {
    pub name: String,
    pub creator: String,
    pub plays: i32,
}

/// Schema for updating an existing note
#[derive(Serialize, Deserialize, Debug)]
pub struct UpdateGameSchema {
    pub name: Option<String>,
    pub creator: Option<String>,
    pub plays: Option<i32>,
}
```

Then add the following code to `handler.rs`:

```rust
// handler.rs

// code for other handlers

pub async fn update_game_handler(
    Path(id): Path<Uuid>,
    State(data): State<Arc<AppState>>,
    Json(body): Json<UpdateGameSchema>,
) -> Result<impl IntoResponse, (StatusCode, Json<serde_json::Value>)> {
    let query_result = sqlx::query_as!(GameModel, r#"SELECT * FROM games WHERE id = $1"#, &id)
        .fetch_one(&data.db)
        .await;

    let game = match query_result {
        Ok(game) => game,
        Err(sqlx::Error::RowNotFound) => {
            let error_response = serde_json::json!({
                "status": "error",
                "message": format!("Game with ID: {} not found", id)
            });
            return Err((StatusCode::NOT_FOUND, Json(error_response)));
        }
        Err(e) => {
            return Err((
                StatusCode::INTERNAL_SERVER_ERROR,
                Json(json!({
                    "status": "error",
                    "message": format!("{:?}",e)
                })),
            ));
        }
    };

    let new_name = body.name.as_ref().unwrap_or(&game.name);
    let new_creator = body.creator.as_ref().unwrap_or(&game.creator);
    let new_plays = body.plays.unwrap_or(game.plays);

    let updated_game = sqlx::query_as!(
        GameModel,
        r#"UPDATE games SET name = $1, creator = $2, plays = $3 WHERE id = $4 RETURNING *"#,
        &new_name,
        &new_creator,
        &new_plays,
        &id
    )
    .fetch_one(&data.db)
    .await
    .map_err(|e| {
        (
            StatusCode::INTERNAL_SERVER_ERROR,
            Json(json!({
                "status": "error",
                "message": format!("{:?}", e)
            })),
        )
    })?;

    let response = json!({
        "status": "success",
        "data": json!({
            "player": updated_game
        })
    });
    Ok(Json(response))
}
```

Everything in this code is similar to what we’ve seen earlier, except for the `unwrap_or()` syntax. Since we’re using the new schema, which has all three fields (name, creator, plays) as optional, we need to handle the `Option` types properly. We learned earlier that `Option` is an enum with two variants: `Some` or `None`. The `unwrap_or()` method returns the value inside the `Option` if it exists (`Some`), or falls back to a default value if it's `None`.

For `name` and `creator` (which are `Option<String>`), we use `as_ref()` because if we pass the actual value instead of the reference, ownership would be transferred, which causes issues like we discussed at the start of this article. `as_ref()` converts `Option<String>` to `Option<&String>`, preventing ownership transfer.

With this, all our handlers are ready. Update the `route.rs` file to add all the handlers there:

```rust
// route.rs

use std::sync::Arc;

use axum::{
    Router,
    routing::{get, patch, post},
};

use crate::{
    AppState,
    handlers::{
        create_game_handler, delete_game_handler, game_list_handler, get_game_handler,
        update_game_handler,
    },
};

pub fn create_router(app_state: Arc<AppState>) -> Router {
    Router::new()
        .route("/api/games", post(create_game_handler))
        .route("/api/games", get(game_list_handler))
        .route(
            "/api/games/{id}",
            get(get_game_handler)
                .delete(delete_game_handler)
                .patch(update_game_handler),
        )
        .with_state(app_state)
}
```

Now let’s test the functionality of our API!

# Testing our Rust CRUD API

We had already tested our POST handler by sending a request to add a game, but let’s add another game so that we can test the GET all games handler too:

```bash
curl -X POST -H "Content-Type: application/json" -d '{"name": "Catan", "creator": "Klaus", "plays": 2}' http://localhost:3000/api/games
```

Now, when you try to get all games, you should see something similar to:

```bash
$  curl -X GET http://localhost:3000/api/games | jq .
{
  "count": 2,
  "notes": [
    {
      "created_at": "2025-09-22T15:26:03.728803Z",
      "creator": "Klaus",                                                                                 "id": "1f426624-3efb-4575-98d1-1f94bc3fe798",
      "name": "Catan",
      "plays": 2
    },
    {
      "created_at": "2025-09-20T16:18:49.102219Z",
      "creator": "Elizabeth Hargrave",
      "id": "d1972ee2-36d0-4794-a8fe-2772c1956707",
      "name": "Wingspan",
      "plays": 23
    }
  ],
  "status": "ok"
}
```

You can also send a request with a game’s specific ID to get the details of just that game:

```bash
$ curl -X GET http://localhost:3000/api/games/1f426624-3efb-4575-98d1-1f94bc3fe798 | jq .
{
  "data": {
    "game": {
      "created_at": "2025-09-22T15:26:03.728803Z",
      "creator": "Klaus",
      "id": "1f426624-3efb-4575-98d1-1f94bc3fe798",
      "name": "Catan",                                                                                    
      "plays": 2
    }
  },                                                                                                  
  "status": "success"
}
```

Let’s now test the UPDATE handler by changing the number of plays for Catan:

```bash
$ curl -X PATCH -H "Content-Type: application/json" -d '{"plays": 5}' http://localhost:3000/api/games/1f426624-3efb-4575-98d1-1f94bc3fe798
{
  "data": {
    "player": {
      "created_at": "2025-09-22T15:26:03.728803Z",
      "creator": "Klaus",
      "id": "1f426624-3efb-4575-98d1-1f94bc3fe798",
      "name": "Catan",
      "plays": 5
    }
  },
  "status": "success"
}
```

Finally, we can send a DELETE request to test the delete handler:

```bash
$ curl -X DELETE http://localhost:3000/api/games/1f426624-3efb-4575-98d1-1f94bc3fe798
{
  "data": {
    "deleted_game": {
      "created_at": "2025-09-22T15:26:03.728803Z",
      "creator": "Klaus",
      "id": "1f426624-3efb-4575-98d1-1f94bc3fe798",
      "name": "Catan",
      "plays": 5
    }
  },
  "message": "Game delete successfully",
  "status": "success"
}
```

And that completes trying out all our API endpoints :)

# Conclusion

Phew! I think this has been the lengthiest technical article I’ve written. I hope you found the content interesting and valuable. I wrote this while learning Rust myself, so some of the code might not be perfectly idiomatic, but I tried my best to explain things in a way that makes sense for beginners. I didn’t find many resources online that explained things in detail about building APIs for someone new to Rust, so I had to do a lot of searching and resort to trial and error to get things to work. Hopefully, this tutorial helps someone in a similar position to mine. Thanks for reading :)