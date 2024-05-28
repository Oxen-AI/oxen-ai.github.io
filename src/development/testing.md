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

# Result<(), OxenError>
