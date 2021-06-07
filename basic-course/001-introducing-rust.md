# Introducing Rust

The Rust language releases a major edition approximately once every three years. 2015, 2018, 2021, and so on. Any crate (that's the name for a Rust package) can be specify which edition it wants to be compiled with when you build your project. So while your crate/binary might be built with Rust 2018, one of your dependencies could be using Rust 2015, and this is totally fine.

In between major releases, the Rust project does rolling releases every six weeks and also nightly releases, so if you want to check how your program performs with some new improvement to the compiler, or test out new stable features we need a way to switch between versions of Rust easily.

That tool is called Rustup.

## install rustup

Rustup is how you manage different Rust versions on your computer.

To install, go to [rustup.rs](https://rustup.rs/) and run the relevant command for your operating system. It will either be a curl command or a .exe depending on your platform of choice. Rust works on Linux, MacOS, and Windows.

```shell
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

This will install a set of tools that you can use to develop and compile Rust programs. The most important component is cargo, Rust's package manager.

## Cargo

Cargo works a lot like npm. The biggest difference is that it doesn't come with a command that will add a crate to your Cargo.toml. To get that, install [cargo-edit](https://crates.io/crates/cargo-edit) which will allow you to use cargo to `add`, `update`, or `rm` dependencies.

Initialize the project we'll be working in using cargo. This will create a new package named `weather-cli` with some files in the `./weather-cli` directory.

```shell
cargo new weather-cli
```

This will create two files: a `Cargo.toml`, and `src/main.rs`.

### Cargo.toml

`Cargo.toml` is the same as a `package.json`. It includes the dependencies our package uses, the version, name, authors, and more.

```toml
[package]
name = "weather-cli"
version = "0.1.0"
authors = ["ChristopherBiscardi <chris@christopherbiscardi.com>"]
edition = "2018"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
```

### src/main.rs

`src/main.rs` includes our first Rust code. Because we didn't specify what kind of package we wanted to create, cargo assumed we wanted a binary. By convention, `src/main.rs` is the filename used as the entrypoint for a binary, just like `index.js` is for JavaScript projects.

```rust
fn main() {
    println!("Hello, world!");
}
```

For comparison, here's the same program above, in JavaScript.

```javascript
function main() {
    console.log("Hello, world!);
}
```

`function` becomes `fn`, and `console.log` becomes `println!`, but otherwise the syntax is very familiar.

`println!` has an exclamation point on the end of it because it is a macro. Think of macros like babel plugins that are built into the Rust language. We can create our own but for now we will only be using the ones provided to us.

## Running our binary

From the root of our project, we can use cargo to run the binary using `cargo run`.

```shell
cargo run
   Compiling weather-cli v0.1.0 (/Users/chris/rust-adventure/weather-cli)
    Finished dev [unoptimized + debuginfo] target(s) in 1.24s
     Running `target/debug/weather-cli`
Hello, world!
```

<Aside>
Because we only have a single binary target in our project at the moment, cargo can guess what we mean and we don't have to explicitly specify which binary we want to run. If we ever have more than a single binary we can specify which one we want to run with `--bin`.

```shell
cargo run --bin weather-cli
```

</Aside>

The `cargo run` output tells us what it did.

First it compiled the `weather-cli` binary at version `0.1.0` from the directory `/Users/chris/rust-adventure/weather-cli`.

Then it finished and gaves us some metadata. We built our binary in debug-mode, which is faster to compile and slower to run, so we see `unoptimized + debuginfo` and how long the compile took.

Finally, cargo ran the binary for us, and gives us the path to the compiled binary that it ran. This path is in `target/`, which is where all of our build artifacts will end up. `target/debug` is the folder for debug artifacts, and `target/release` would be where our release artifacts would be stored. Since the Rust compiler is an incremental compiler, the target directory is also where intermediary compilation artifacts are stored. This is why the second compile is faster than the first.

Our program used a `println!` to print some output to stdout, and we also see that `Hello, world!` was printed out when our binary ran.

We can run the built binary ourselves by calling it directly at the path cargo gave us. (note: on windows the path will look slightly different, and it will end in .exe).

```shell
./target/debug/weather-cli
```

If we wanted to compile in release mode, we could use.

```shell
cargo run --release
```

## Lockfiles

After we built the binary (by doing `cargo run`), cargo created a `Cargo.lock` file with the exact details of the dependencies that were used. Our `Cargo.lock` doesn't have much in it because we don't have many dependencies, but it will grow as we add them. This file is maintained by Cargo and should not be manually edited.

```toml
# This file is automatically @generated by Cargo.
# It is not intended for manual editing.
[[package]]
name = "weather-cli-test-one"
version = "0.1.0"
```
