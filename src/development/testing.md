# Testing

We are all smart software engineers, but when it comes to entering a new codebase we all want confidence that making a change doesn't have a cascading effect. It is important to make sure that turning off the (proverbial) lights in the kitchen 💡 doesn't make the roof collapse 🏠.

Luckily each command within Oxen has a well defined interface, and each command can be tested independently. 

For example:

```rust
// Initialize Repo
let repo = command::init("./test_repo")?;
// Add File
command::add(&repo, &"hello.txt")?;
// Commit File
command::commit(&repo, &"add hello.txt")?;
```

We chain these commands together into a sequence of integration and unit tests to make sure the end to end system works as expected.

# Writing Tests

The best place to reference when looking at tests within Oxen are the `lib/src/command` modules themselves. You'll find some familiar names within the `command::` namespace.

For example:

- [command::init](https://github.com/Oxen-AI/Oxen/blob/main/src/lib/src/command/init.rs)
- [command::add](https://github.com/Oxen-AI/Oxen/blob/main/src/lib/src/command/add.rs)
- [command::commit](https://github.com/Oxen-AI/Oxen/blob/main/src/lib/src/command/commit.rs)

All tests for these commands are found below their respective module. Let's look at an example command and break down the different parts of the test.

```rust
#[cfg(test)]
mod tests {
    // ... include necessary modules

    #[test]
    fn test_command_init() -> Result<(), OxenError> {
        test::run_empty_dir_test(|repo_dir| {
            // Init repo
            let repo = command::init(repo_dir)?;

            // Init should create the .oxen directory
            let hidden_dir = util::fs::oxen_hidden_dir(repo_dir);
            let config_file = util::fs::config_filepath(repo_dir);
            assert!(hidden_dir.exists());
            assert!(config_file.exists());

            Ok(())
        })
    }
}
```

First you will notice that the tests are within a `mod tests` block. This is a Rust feature that allows you to group tests together within a particular module.

In order to run all the tests within a particular command module you can run:

```bash
cargo test --lib command::init
```

This will run all the tests within the `command::init` module.

# Returning Errors

You will notice that all the tests return `Result<(), OxenError>`. This means they will catch any errors that might occur when running different command.

The `OxenError` is a custom error type that is defined in the [lib/src/error.rs](https://github.com/Oxen-AI/Oxen/blob/main/src/lib/src/error.rs) file. It is a simple enum that represents an error that can occur in Oxen. When you unwrap `?` a function that returns a `Result<(), OxenError>` you will receive the error and the test will fail.

# Setup & Teardown

Next you will see that most tests are wrapped in a closure defined in our [test.rs](https://github.com/Oxen-AI/Oxen/blob/main/src/lib/src/test.rs) file.

```rust
test::run_empty_dir_test(|repo_dir| {
    // ... your test code here

    Ok(())
})
```

These closures takes care of a lot of the boiler plate around setting up a test directory, and deleting it after the test is run. 

For example `run_empty_dir_test` will pass a unique directory to the closure, and delete it when finished. This way we can run all the isolated tests in parallel and not worry about leaking files from one test impacting another.

There are many other helper functions you can use to setup and teardown your tests, including populating repositories with sample data, and setting up remote repositories. See the full list in the [test.rs](https://github.com/Oxen-AI/Oxen/blob/main/src/lib/src/test.rs) file.

# Running All Tests

Make sure your server is running on the default port and host, then run

*Note:* tests open up a lot of file handles, so limit num test threads if running everything.

You an also increase the number of open files your system allows ulimit before running tests:

```
ulimit -n 10240
```

```
cargo test -- --test-threads=$(nproc)
```

It can be faster (in terms of compilation and runtime) to run a specific test. To run a specific library test:

```
cargo test --lib test_get_metadata_text_readme
```

To run with all debug output and run a specific test

```
env RUST_LOG=debug,liboxen=debug,integration_test=debug cargo test -- --nocapture test_command_push_clone_pull_push
```

To set a different test host you can set the `OXEN_TEST_HOST` environment variable

```
env OXEN_TEST_HOST=0.0.0.0:4000 cargo test
```
