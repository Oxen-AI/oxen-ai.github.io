# Coding Guidelines

TODO: Look up some basic rust ones

https://doc.rust-lang.org/nightly/style-guide/

# fmt & clippy

Before checking in a PR, please make sure to run `cargo fmt` and `cargo clippy`. This will format your code and check for errors.

```bash
cargo fmt
cargo clippy --fix --allow-dirty
```

# PathBuf & Path

Use Path and PathBuf over String and &str when referencing paths. This is because PathBuf is a struct that represents a path and is more powerful than a string. For example, it makes sure the paths are cross platform (windows and unix) and allows you to check if a path is a file or directory. PathBuf also has other useful methods to get the file name, directory name, etc.