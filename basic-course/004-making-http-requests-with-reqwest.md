# 004 Making HTTP Requests with reqwest

```rust
let client = reqwest::blocking::Client::new();

let response = client
    .get("https://api.waqi.info/search/")
    .query(&[("token", api_token), ("keyword", args)])
    .send()
    .expect("a successful request")
    .json::<serde_json::Value>()
    .expect("expected the body to be json");
```

Today we add a few dependencies to our `Cargo.toml`: `reqwest`, `serde`, and `serde_json`.

`serde` is the standard for serializing and deserializing Rust data structures. You will see it used all across the Rust ecosystem. `serde_json` provides efficient, flexible, safe ways of converting JSON between three different representations: text data (a json string), untyped, and strongly typed.

The untyped representation will mean accepting any possible JSON and is what we'll be using today as we do not yet know what to expect from our API responses.

```toml
[package]
name = "weather-cli"
version = "0.1.0"
authors = ["ChristopherBiscardi <chris@christopherbiscardi.com>"]
edition = "2018"

[dependencies]
reqwest = { version = "0.11.3", features = ["json", "blocking"] }
serde = "1.0.126"
serde_json = "1.0.64"
```

The `reqwest` dependency is specified differently than the other two. This is because we need to take advantage of the fact that Cargo lets us specify which features we want from a particular crate. In this case, we'll be using the `json` and `blocking` features from the `reqwest` crate. Cargo features allow crate authors to expose options for users to conditionally include pieces of functionality they need.

The `json` feature is being used because we'll be serializing http responses to JSON, and the `blocking` feature is being used because we haven't covered async/await in Rust and won't be in this sequence. The `blocking` feature allows us to put off learning about async Rust.

## Creating a Client

Firstly we need to create a new `reqwest` `Client` that we can use to make HTTP requests. We do this by calling the `new` method from the `Client` struct in the `reqwest::blocking` module.

```rust
let client = reqwest::blocking::Client::new();
```

The [`get`](https://docs.rs/reqwest/0.11.3/reqwest/blocking/struct.Client.html#method.get) function on `client` returns a [`RequestBuilder`](https://docs.rs/reqwest/0.11.3/reqwest/blocking/struct.RequestBuilder.html) struct.

```rust
client.get("https://api.waqi.info/search/")
```

The builder pattern (that is, calling more functions to add options before executing whatever it is we need to do), is very common in Rust and it's what we'll do here to add query params. [`.query`](https://docs.rs/reqwest/0.11.3/reqwest/blocking/struct.RequestBuilder.html#method.query) returns a new `RequestBuilder` with the additional options.

```rust
client
    .get("https://api.waqi.info/search/")
    .query(&[("token", api_token), ("keyword", args)])
```

Let's take a second to talk about the arguments we just pased to `.query`. We are passing in a reference to an `array` of two tuples.

An `array` in Rust isn't what you'd think of as an array in other languages like JavaScript. The JavaScript Array is called a [`Vec`](https://doc.rust-lang.org/std/vec/struct.Vec.html) in Rust. An `array` in Rust is a fixed-size collection of similar elements. So we have an `array` of length 2, which is made up of tuples of size 2, in our example.

While the elements of an `array` must all have the same type, that is not true for tuples, whose elements can have different types. Tuples are denoted by parens, and can have 0 or more elements. `()`, `("one")`, and `("one", 2)` are all valid tuples.

Finally we're using `&` to convert our `array` into a `slice`. This means we aren't passing in the `array` itself but rather a shared view into that `array`. This idea of passing around shared references when we don't need to mutate data will become more natural as you write more Rust.

Finally we need to do something with our `RequestBuilder`. We can use [`.send`](https://docs.rs/reqwest/0.11.3/reqwest/blocking/struct.RequestBuilder.html#method.send) which constructs the HTTP request and sends it, returning a `Result<Response>`.

```rust
client
    .get("https://api.waqi.info/search/")
    .query(&[("token", api_token), ("keyword", args)])
    .send()
```

Since we're getting back a `Result` we can use `.expect` again. This will panic if the HTTP request fails and crash our program, but that's ok for us for now.

```rust
client
    .get("https://api.waqi.info/search/")
    .query(&[("token", api_token), ("keyword", args)])
    .send()
    .expect("a successful request")
```

Remember that `expect` will also unwrap our `Result`, leaving us with a `Response` type. `Response` has a function called [`json`](https://docs.rs/reqwest/0.11.3/reqwest/blocking/struct.Response.html#method.json) which will try to deserialize the HTTP response using serde.

`.json` can deserialize into many different types. Any that implement serde's [`Deserialize` trait](https://docs.serde.rs/serde/trait.Deserialize.html) even, which is a lot! So we have to tell it which type we want it to deserialize into.

`serde_json` offers the [`Value` enum type](https://docs.serde.rs/serde_json/value/enum.Value.html), which can hold any valid JSON, so we'll use that.

To give a type to a function, we can use what is affectionately called the "turbofish" because the syntax [looks a bit like a fish](https://turbo.fish/) (`::<>`) and people like cute names. We can put our type in between the angled brackets and `.json` will know what type to deserialize into.

```rust
client
    .get("https://api.waqi.info/search/")
    .query(&[("token", api_token), ("keyword", args)])
    .send()
    .expect("a successful request")
    .json::<serde_json::Value>()
```

Finally, `.json` returns a `Result<serde_json::Value, reqwest::Error>`, so we can `.expect` again to unwrap the value if it succeeds and panic if it fails.

Remember, panics are unrecoverable so as you write more Rust use them sparingly.

```rust
let response = client
    .get("https://api.waqi.info/search/")
    .query(&[("token", api_token), ("keyword", args)])
    .send()
    .expect("a successful request")
    .json::<serde_json::Value>()
    .expect("expected the body to be json");
```

and finally we can `dbg!` the response, which will print the entire `serde_json::Value` struct with the deserialized JSON out to the console.

```rust
let client = reqwest::blocking::Client::new();

let response = client
    .get("https://api.waqi.info/search/")
    .query(&[("token", api_token), ("keyword", args)])
    .send()
    .expect("a successful request")
    .json::<serde_json::Value>()
    .expect("expected the body to be json");

dbg!(response);
```

You can now run

```shell
API_TOKEN=<my-token> cargo run -- san francisco
```

and you will see your output look something like this. `Object`, `Array`, `String`, `Number` are all variants of the `serde_json::Value` enum, which means their fully qualified names are `serde_json::Value::Object`.

```rust
Object({
    "data": Array([
        Object({
            "aqi": String("85"),
            "station": Object({
                "country": String("MX"),
                "geo": Array([Number(19.260413888889),
                Number(-99.645613888889)]),
                "name": String("Ceboruco,
                Toluca,
                Mexico"),
                "url": String("mexico/toluca/ceboruco")
            }),
            "time": Object({
                "stime": String("2021-05-31 23:00:00"),
                "tz": String("-05:00"),
                "vtime": Number(1622520000)
            }),
            "uid": Number(3547)
        }),
    ]),
    "status": String("ok")
})
```

## Quiz

Here is a link to [the reqwest docs](https://docs.rs/reqwest/0.11.3/reqwest/blocking/struct.RequestBuilder.html) for `RequestBuilder`. Let's say you wanted to add a user agent header to the reqwest (some services, like Discord, require you to set a unique user agent for your client).

How would you add the user agent header?

### Answers

You can use the `.header` function on `RequestBuilder` to add additional headers.

```rust
.header(reqwest::header::USER_AGENT, "my weather app")
```

Turning the program into this:

```rust
let client = reqwest::blocking::Client::new();

let response = client
    .get("https://api.waqi.info/search/")
    .query(&[("token", api_token), ("keyword", args)])
    .header(reqwest::header::USER_AGENT, "my weather app")
    .send()
    .expect("a successful request")
    .json::<serde_json::Value>()
    .expect("expected the body to be json");

dbg!(response);
```
