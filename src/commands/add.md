# oxen add

After initializing a repository, you can add files to it using the `oxen add` command.

```bash
oxen add <file>
```

This is the workhorse of oxen that does most of the compute. Under the hood, `oxen add` does a few operations:

* Hashes the file(s) and directory structure
* Computes any additional metadata about the file
  * Sizes
  * File Types
  * Schemas (for JSON, CSV, etc.)
* Copies a version of the file into the content addressable `versions` store
* Adds a record to the `staged` index

# Under the hood

In order to see what this looks like on disk, let's create a few files and directories, then add them to a fresh repository.

```bash
mkdir my-project
cd my-project
oxen init
```

TODO: