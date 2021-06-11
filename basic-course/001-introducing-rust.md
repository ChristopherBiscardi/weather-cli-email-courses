# Introducing Rust

## Install Rustup

In between major Editions, the Rust project does rolling releases every six weeks and also nightly releases, so if you want to check how your program performs with some new improvement to the compiler, or test out new stable features we need a way to switch between versions of Rust easily.

Rustup is how you manage different Rust versions on your computer.

To install, go to [rustup.rs](https://rustup.rs/) and run the relevant command for your operating system. It will either be a curl command or a .exe depending on your platform of choice. Rust works on Linux, MacOS, and Windows.

```shell
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

This will install a set of tools that you can use to develop and compile Rust programs. The most important component is cargo, Rust's package manager.

<Aside>
The Rust language releases a major edition approximately once every three years. 2015, 2018, 2021, and so on. Any crate (that's the name for a Rust package) can specify which edition it wants to be compiled with when you build your project. So while your crate/binary might be built with Rust 2018, one of your dependencies could be using Rust 2015, and this is totally fine.
</Aside>

## Cargo

Cargo works a lot like npm. The biggest difference is that it doesn't come with a command that will add a crate to your Cargo.toml. To get that:

Install [cargo-edit](https://crates.io/crates/cargo-edit).

`cargo-edit` will allow you to use cargo to `add`, `update`, or `rm` dependencies.

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
Because we only haven't specified any targets of our own in `Cargo.toml` *and* we have the conventional binary entry point `src/main.rs` that means that we have a single binary target in our project at the moment. Cargo can guess what we mean and we don't have to explicitly specify which binary we want to run (again, because we only have one to choose from). If we ever have more than a single binary we can specify which one we want to run with `--bin`.

```shell
cargo run --bin weather-cli
```

</Aside>

The `cargo run` output tells us what it did.

First it compiled the `weather-cli` binary at version `0.1.0` from the directory `/Users/chris/rust-adventure/weather-cli`.

Then it finished and gave us some metadata. We built our binary in debug-mode, which is faster to compile and slower to run, so we see `unoptimized + debuginfo` and how long the compile took.

Finally, cargo ran the binary for us, and gave us the path to the compiled binary that it ran. This path is in `target/`, which is where all of our build artifacts will end up. `target/debug` is the folder for debug artifacts, and `target/release` would be where our release artifacts would be stored. Since the Rust compiler is an incremental compiler, the target directory is also where intermediary compilation artifacts are stored. This is why the second compile is faster than the first.

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

---

## Quiz

We created a file called `src/main.rs` and a function in that file called `fn main`. What happens if either of these are called something else?

### Answers

#### No main.rs file

The name and location of the `main.rs` file is important, but only because we haven't configured which file to use as the entrypoint for the binary target. If we don't specify the target details in `Cargo.toml` _and_ we don't have the default `src/main.rs` file, then we see this error message from the compiler when we build.

```
❯ cargo build
error: failed to parse manifest at `/Users/chris/weather-cli/Cargo.toml`

Caused by:
  no targets specified in the manifest
  either src/lib.rs, src/main.rs, a [lib] section, or [[bin]] section must be present
```

This message is telling us that we need at least one library or binary target to be able to build the project. If we don't want to use `src/main.rs`, we can specify the binary by adding a [`[[bin]]` field](https://doc.rust-lang.org/cargo/reference/cargo-targets.html#binaries) in our `Cargo.toml`

```
[[bin]]
name = "weather-cli"
path = "src/my-binary-file.rs"
```

This is also how you'd specify a number of other configuration options for how to build a specific target, such as the name of the binary that gets built.

#### No `main` function

If we rename the main function in `main.rs` to something nonsensical like `fn asfasf()` then when we try to build we'll see this error message.

```
❯ cargo build
   Compiling thingd v0.1.0 (/Users/chris/weather-cli)
error[E0601]: `main` function not found in crate `weather-cli`
 --> src/main.rs:1:1
  |
1 | / fn asfasf() {
2 | |     println!("Hello, world!");
3 | | }
  | |_^ consider adding a `main` function to `src/main.rs`

error: aborting due to previous error

For more information about this error, try `rustc --explain E0601`.
error: could not compile `thingd`

To learn more, run the command again with --verbose.
```

The most important piece of the error message is this line:

> consider adding a `main` function to `src/main.rs

This error occurs because we _must_ have a function called `main` in the binary entrypoint file. It is a convention enforced by the compiler.

In the future when we get into async Rust and other higher level macros, this error can pop up if you've misconfigured a macro that wraps the main function.
