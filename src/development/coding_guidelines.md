# Coding Guidelines

TODO: Add some basic rust ones

https://doc.rust-lang.org/nightly/style-guide/

## fmt & clippy

Before checking in a PR, please make sure to run `cargo fmt` and `cargo clippy`. This will format your code and check for errors.

```bash
cargo fmt
cargo clippy --fix --allow-dirty
```

## Try to avoid .clone() if possible

Pass values by reference or by value instead of cloning them unless absolutely necessary. Cloning can be expensive, especially for large structs or strings.

## Use PathBuf and Path over String and str

When referencing file system paths, use Path and PathBuf over String and &str. This is because PathBuf is a struct that represents a path and is more powerful than a raw string. For example, it makes sure the paths are cross platform (windows and unix) and allows you to check if a path is a file or directory. PathBuf also has other useful methods to get the file name, directory name, components, etc.

## Use `impl AsRef` where possible

As function parameters, instead of taking in a `&Path`, `&str`, `PathBuf` or `String` take in a `impl AsRef<Path>` or `impl AsRef<str>`. This way the consumer can decide whether or not they want the value to be borrowed or passed in by reference, and does not have to make sure the value is a reference.

This makes it much is easier and more flexible for external consumers.

TODO: Examples of signatures and external consumers

## Be cognizant of loading too much of the Merkle Tree

The `CommitMerkleTree` struct lets you load subsets of the merkle tree and skip directly to dir nodes. It also has functionality to only load 1 or 2 levels deep (avoiding deep recursion).

Make sure you are only loading what you need to get the information you need to return.

For example, if you just need the size of a directory, you don't need to load it's children.

```rust
let load_recursive = false;
let node = CommitMerkleTree::from_path(repo, commit, path, load_recursive)?;
```

If you need all the files in a directory, and you don't want all the levels below it, you can specify the depth to load.

```rust
// This will load the VNodes and the FileNode/DirNode children of the VNodes
let node = CommitMerkleTree::read_depth(repo, hash, 2)?;
```