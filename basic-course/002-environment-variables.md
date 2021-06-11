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

---

## Quiz

Semicolons are important in other languages for seemingly arbitrary reasons. What happens if you remove the semicolon from the `dbg!` call in our program? What do you think the error message is telling you?

### Answers

When we remove the semicolon from the `dbg!` macro we get this error message.

```
❯ cargo build
   Compiling weather-cli v0.1.0 (/Users/chris/weatcher-cli)
error[E0308]: mismatched types
 --> src/main.rs:4:5
  |
1 | fn main() {
  |           - expected `()` because of default return type
...
4 |     dbg!(api_token)
  |     ^^^^^^^^^^^^^^^ expected `()`, found struct `String`
  |
  = note: this error originates in a macro (in Nightly builds, run with -Z macro-backtrace for more info)

error: aborting due to previous error

For more information about this error, try `rustc --explain E0308`.
error: could not compile `thingd`

To learn more, run the command again with --verbose.
```

There are a couple of key insights here.

1. The default return value of a function is `()`
2. Everything in Rust is either a Statement or an Expression
3. `dbg!` returns the value of the expression we pass it
4. Semicolons turn Expressions into Statements

## 1. The default return value of a function is `()`

`()` is pronounced "unit" and is a 0 element tuple. If we don't specify a return type for our functions, they return `()` by default. The type unit `()` and the only value that can satisfy that type `()` are spelled the same way. This means if a function's return type is `()`, then the only value we can return at the end of our function is `()`.

By comparison in JavaScript if a function returns the type "Number", then it would return values like: `1`, `2`, `3`, etc.

## 2. Everything in Rust is either a Statement or an Expression

Rust is primarily an Expression based language. This means that inside of a block there is always a return value from that block. This even applies to `if`.

In JavaScript `if` is a statement, and thus doesn't return a value. This leads us into using hacks like ternary operators, or `mything && MyComponent`.

In Rust, we can return values from an if expression and put them into a variable as such.

```rust
let y = if 12 * 15 > 150 {
    "Bigger"
} else {
    "Smaller"
};
```

We could put this into a function that returns a `String`. Notice how we removed the variable assignment as well as the semi-colon. The last value in an expression will be returned.

```rust
fn test_a_number() -> String {
    if 12 * 15 > 150 {
        "Bigger".to_string()
    } else {
        "Smaller".to_string()
    }
}
```

In this case, that means `"Bigger".to_string()` is returned from one branch of the if expression, and `"Smaller".to_string()` is returned from the other branch of the if expression.

This means that overall the if expression returns a `String`, and since the if expression is the last expression in the function body, the function returns a `String` as well.

## 3. `dbg!` returns the value of the expression we pass it

One unique feature of the `dbg!` macro is that since it is use for debugging purposes, the value returned from it is the value of the expression we pass it. It's as if the `dbg!` macro wasn't even there.

In the following example, we print out the result of `dbg!(2) == 2`, which is the same as `2 == 2`, to prove that `dbg` passes the value through.

```rust
fn main() {
    println!("{}", dbg!(2) == 2);
}
```

## 4. Semicolons turn Expressions into Statements

So to recap:

- Functions return the `()` type by default
- Most things in Rust are expressions
- The last value in an expression is returned from that expression
- `dbg!` returns whatever we give it, as is.

That means we're left with a main function with `dbg!` in the last position, which returns whatever we gave it. In this case, that was the api token: a `String`.

This lets us read the error message we got from the Rust compiler. First, the compiler expected `()` because that's a function's default return type.

```rust
  |
1 | fn main() {
  |           - expected `()` because of default return type
```

Then, on line 4, the compiler expected the `()` value because the default return type of the function told it to. Instead, it found that we are returning a `String` because `dbg!` returns whatever we give it, and `dbg!` is in the last position in the function body, which means whatever value it is, is what gets returned from the function.

```rust
4 |     dbg!(api_token)
  |     ^^^^^^^^^^^^^^^ expected `()`, found struct `String`
  |
```

So the question becomes how do we discard the `dbg!` value and the answer is the semicolon: `;`.

A semicolon is an operator that takes an expression, evaluates it, then discards the result. In place of the discarded result, we get `()`, which is the return type we need for the `main` function.

This is why the semicolon is important. It takes our `dbg!` expression and evaluates it which lets us print to the console, while also letting us return `()`.
