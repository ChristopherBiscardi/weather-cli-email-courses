# 002 Environment variables in Rust

The Rust standard library is provided as the `std` crate. In practice this means `std` behaves much like any other package, except that it's always around and you don't have to install it. This also means you can create programs without the `std` crate, which is called `no_std`.

For our use case, the Rust standard library provides a set of functionality in the `std::env` module. Modules are containers of 0 or more functions, types, other modules, and more. Specifically in our case, the `std::env` module defines [a `var` function](https://doc.rust-lang.org/std/env/fn.var.html) for getting environment variables.

Here we declare our first variable, `api_token` using the word `let`. Variables in Rust are immutable by default, so this value will not change for the rest of our program.

```rust
fn main() {
    let api_token = std::env::var("API_TOKEN").expect("expected there to be an api token");
}
```

Rust doesn't have the concept of `null` or `undefined`. What it has instead are data types that can carry information about whether a function succeeded or failed. `std::env::var` uses the `Result` type for this because it can succeed or fail to get a value from an environment variable with the specified name.

The names for success and failure are `Ok` and `Err`.

`Result` is an enum with `Ok` and `Err` variants.

In Rust, we use `Result` quite a bit to handle errors.
