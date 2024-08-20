# Coding Guidelines

TODO: Look up some basic rust ones

https://doc.rust-lang.org/nightly/style-guide/

# fmt & clippy

Before checking in a PR, please make sure to run `cargo fmt` and `cargo clippy`. This will format your code and check for errors.

```bash
cargo fmt
cargo clippy --fix --allow-dirty
```

# Use PathBuf and Path over String and str

When referencing file system paths, use Path and PathBuf over String and &str. This is because PathBuf is a struct that represents a path and is more powerful than a raw string. For example, it makes sure the paths are cross platform (windows and unix) and allows you to check if a path is a file or directory. PathBuf also has other useful methods to get the file name, directory name, components, etc.

# Use `impl AsRef` where possible

As function parameters, instead of taking in a `&Path`, `&str`, `PathBuf` or `String` take in a `impl AsRef<Path>` or `impl AsRef<str>`. This way the consumer can decide whether or not they want the value to be borrowed or passed in by reference, and does not have to make sure the value is a reference.

This makes it much is easier and more flexible for external consumers.

TODO: Examples of signatures and external consumers