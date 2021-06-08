# 002 Environment variables in Rust

In this example we declare our first variable, `api_token` using the word `let`. Variables in Rust are immutable by default, so this value will not change for the rest of our program.

```rust
fn main() {
    let api_token = std::env::var("API_TOKEN")
          .expect("expected there to be an api token");
}
```

Conceptually, in JavaScript, you can think of `std` like it's a package, `env` like it's a specific group of exports in that package, and `var` as a function in `env`.

```javascript
import { var } from 'std/env';

function main() {
    const api_token = var("API_TOKEN");
    console.log({api_token});
}
```

The Rust standard library is provided as the `std` crate. In practice this means `std` behaves much like any other package, except that it's always around and you don't have to install it. This also means you can create programs without the `std` crate, which is called `no_std`.

`<image>std modules: env, fs, path, etc, with ... for most and (fn var) in env`

For our use case, the Rust standard library provides a useful set of functionality in the `std::env` module. Modules are containers of 0 or more functions, types, other modules, and more. Specifically in our case, the `std::env` module defines [a `var` function](https://doc.rust-lang.org/std/env/fn.var.html) for accessing environment variables.

`<image>std:env module with functions. bullet point is fn`

Rust doesn't have the concept of `null` or `undefined`. What it has instead are data types that can carry information about whether a function succeeded or failed. `std::env::var` uses the `Result` type for this because it can succeed or fail to get a value from an environment variable with the specified name.

`image: bubbles with Ok(4) or Err(ParseIntError("msg"))` to show Results.

## The Result Type

The names for success and failure are `Ok` and `Err` and you can think of them as "wrapping" your desired value. If `std::env::var` is successful, we will get the value of the environment variable inside of the `Ok`, like `Ok("token")`. If `std::env::var` fails, for example if the environment variable isn't set in the environment, we will get an error wrapped in `Err` like `Err(NotPresent)`.

`Result` is an enum with `Ok` and `Err` variants. The definition then looks something like this:

```rust
enum Result<THING_WE_WANT, ERROR_THAT_COULD_HAPPEN> {
    Ok(THING_WE_WANT),
    Err(ERROR_THAT_COULD_HAPPEN)
}
```

So that makes the `Result` type we're getting back from `std::env::var`: `Result<String, VarError>`. If successful we'll get back a [`String`](https://doc.rust-lang.org/std/string/struct.String.html) and if unsuccessful we'll get back a [`VarError`](https://doc.rust-lang.org/std/env/enum.VarError.html).

In Rust, we use `Result` quite a bit to handle errors.

## dbg!

Let's take a short detour to examine the return value of `std::env::var` in our program. Using `dbg!` we can print out `api_token` to the console and inspect its value for ourselves.

```rust
fn main() {
    let api_token = std::env::var("API_TOKEN");
    dbg!(api_token);
}
```

The `dbg` macro is different than the `println!` macro in a couple important ways and is specifically used for debugging your own code. `println!` prints exactly what you ask it to, to stdout while `dbg` includes a bunch of additional debugging information.

So if `API_TOKEN` is not set in the environment, we will see the source file name and line number that the `dbg!` is called on. This allows us to the file and line a dbg log came from.

After the filename `dbg!` will print out the expression we gave it and then the value that expression evaluates to. In this case we passed in the `api_token` variable name and the value is the `Err(NotPresent)` we talked about earlier.

```
[src/main.rs:3] api_token = Err(
    NotPresent,
)
```

A successful execution of `std::env::var` will output the following using `dbg!`.

```
[src/main.rs:3] api_token = Ok(
    "my-token",
)
```

## .expect

In this case, we want the program to stop if the environment variables isn't set, which means we can use [`.expect`](https://doc.rust-lang.org/std/result/enum.Result.html#method.expect) on the `Result` that we get back from `std::env::var`. `.expect` will do two things for us.

First, if we get back an `Ok("some-token")` value from `std::env::var`, `.expect` will unwrap it and give us just the `String`. This means `api_token` will be a `String` and not a `Result` type.

Secondly, if we get an `Err` back from `std::env::var`, `.expect` will panic and immediately crash our program, printing the error to the console.

This is a bit harsh. panic isn't typically something we want to do when handling errors as there is no way to recover from it. In this case it works for what we want to accomplish.

What we end up with is the token from the environment variable in the `api_token` variable that we can then use to make requests to the waqi service.

## Getting a token

To get a token we can use with our project, go to [aqicn.org](http://aqicn.org/data-platform/token/#/) and agree to the terms of service by entering your email address. They will give you a token to access the API we are going to use.

After getting a token, you can use your favorite way to get an environment variable into your shell environment. My personal favorite way is using [direnv](https://direnv.net/), but I'll show you how to do it inline here.

On unix systems (Mac OS, Linux, etc) you can define the environment variable before running our binary like this. Replace `<my-token>` with your token.

```
API_TOKEN=<my-token> cargo run
```

On Windows

TODO

To test to make sure our program is getting access to the environment variable, we can use the `dbg!` macro. Your entire program should now look like this.

```rust
fn main() {
    let api_token = std::env::var("API_TOKEN")
          .expect("expected there to be an api token");
    dbg!(api_token);
}
```

and running the program will print out the api token.
