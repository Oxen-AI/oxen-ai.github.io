# Repositories

When we talk about data in Oxen, we usually talk about "Repositories". A Repository lives within your working directory of data in a hidden `.oxen` directory. You can think of a Repository as a series of snapshots of your data at any given point in time. 

TODO: Image

Each snapshot contains a "mini filesystem" representing all the files and folders in that snapshot. The each mini filesystem is represented by a [commit](domains/commits.md), and is stored in the `.oxen` directory so that we can return to it at any point in time.

## LocalRepository

Since all of the data for all of the versions is simply stored in a hidden subdirectory, the first object we introduce is the `LocalRepository`. This object simply represents the `path` to the repository so that we know where to look for subsequent objects.

[src/lib/src/model/repository/local_repository.rs](https://github.com/Oxen-AI/Oxen/blob/main/src/lib/src/model/repository/local_repository.rs)

```rust
pub struct LocalRepository {
    pub path: PathBuf,
    // Optional remotes to sync the data to
    remote_name: Option<String>,
    pub remotes: Vec<Remote>,
}
```

Whenever starting down a code path within the CLI the first thing we do is find where the `.oxen` directory is and instantiate our `LocalRepository` object.