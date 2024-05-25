# Commits

If you are familiar with `git` the concept of a `commit` and `branch` should be very familiar. What you may not have done is look under the hood as to how they are stored. In Oxen, many of the concepts are similar.

A commit is a checksum or [hash value](optimizations/hashing.md) representing all the files is within a specific version. You may recognize them as a string of hexadecimal characters  (0–9 and a–f) looking something like `4b27c4b16fb736a4`.

Run `oxen log` within your Oxen repository and you will see the initial commit.

```
commit 4b27c4b16fb736a4

Author: ox
Date:   Thursday, 09 May 2024 22:29:00 +00

    Initialized Repo 🐂
```

You will see these hashes all over the place in Oxen and can use them as pointers to get to specific versions.

# Commits Database

In order to understand the Commit object better, let's look at how it is stored on disk. Again we are going to peek in the `.oxen` directory as our starting point. Specifically at the `commits` directory.

```
$ ls .oxen/commits

000008.sst       000014.log       IDENTITY         LOG              OPTIONS-000012
000013.sst       CURRENT          LOCK             MANIFEST-000015  OPTIONS-000017
```

This entire directory is the commits database. Under the hood Oxen stores most of it's data with [RocksDB](https://rocksdb.org/). This is a fast key-value store that makes it fast to insert and read data without loading the entire database into memory. Specifically we use the rust bindings for RocksDB found in [this repository](https://docs.rs/rocksdb/latest/rocksdb/).

These files are hard to inspect on their own, so we can use the `oxen db` command to inspect the commits database. The `oxen db list` command will print out all the commits in the database.

```
$ oxen db list .oxen/commits
```

You'll see this prints the entire key value store as tab separated values.

```
2c610ae8e424a4c8	{"id":"2c610ae8e424a4c8","parent_ids":["440b54a690b44fd7"],"message":"Add hello.txt and world.txt","author":"oxbot","email":"oxbot@oxen.ai","root_hash":"cf2c5e5f057b589230654260d07fa7c3","timestamp":"2024-05-23T00:40:20.555747Z"}
440b54a690b44fd7	{"id":"440b54a690b44fd7","parent_ids":[],"message":"Initialized Repo 🐂","author":"oxbot","email":"oxbot@oxen.ai","root_hash":"99aa06d3014798d86001c324468d497f","timestamp":"2024-05-23T00:40:02.047406Z"}
a36e6239a1ab49d4	{"id":"a36e6239a1ab49d4","parent_ids":["2c610ae8e424a4c8"],"message":"Update hello.txt","author":"oxbot","email":"oxbot@oxen.ai","root_hash":"7121f5302a90ea338f129ca169a39739","timestamp":"2024-05-23T00:43:53.303893Z"}
```

If you want to get a specific commit, you can use the `oxen db get` command. For example, to get the commit `440b54a690b44fd7`, you can run the following command.

```
$ oxen db get .oxen/commits/ 2c610ae8e424a4c8 | jq
```

```
{
  "id": "2c610ae8e424a4c8",
  "parent_ids": [
    "440b54a690b44fd7"
  ],
  "message": "Add hello.txt and world.txt",
  "author": "oxbot",
  "email": "oxbot@oxen.ai",
  "timestamp": "2024-05-23T00:40:20.555747Z"
}
```

Ah, there is a nice beautiful commit. You can see that the raw values are simply stored in the db as json. There is a list of parent commit ids, a message, the author, the email, and the timestamp.

# Commit Metadata

Each commit stores data about the user that committed it. The user data is read from the global `~/.config/oxen/user_config.toml` file. This is a toml file that contains the user's name and email.

You can set your user info with the `oxen config` command.

```bash
$ oxen config --name 'Bessie' --email 'bessie@your_email.com'
```

```bash
$ cat ~/.config/oxen/user_config.toml
```

```
name = "Bessie"
email = "bessie@your_email.com"
```

It also contains the timestamp of the commit, and a user provided message. All of these pieces of data are used in computing the commit id, which is a unique representation of the data in this commit.

# Commit Id (Hash)

Each commit has a unique id (hash) that can be verified to ensure the integrity of the data in this commit. It is a combination of the data within all the files of the commit, the user data, timestamp, and the message.

TODO: Link to the code that computes the hash.

# 🌲 Commit Merkle Tree

TODO: Talk about merkle tree. Maybe in different section

As you can see, each commit has a list of parent commit ids. This is used to create a [commit tree](https://en.wikipedia.org/wiki/Commit_tree). The root commit will have no parent commits.

Our first couple of examples will have a linear commit tree with one parent per commit. This is the most common case when getting started. But as you will see later with merge commits, commits can have multiple parent commits when combining work from multiple branches.
