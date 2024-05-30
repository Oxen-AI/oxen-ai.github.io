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

If you want to get a specific commit, you can use the `oxen db get` command. For example, to get the commit `2c610ae8e424a4c8`, you can run the following command.

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

Ah, there is a nice beautiful commit object. You can see that the raw values are simply stored in the db as json. There is a list of parent commit ids, a message, the author, the email, and the timestamp.

# Commit Metadata

All of the metadata within a commit object is important for computing it's `id`. The id can be used to verify the integrity of the data within a commit. More on this later.

The first piece of metadata is the user that made the commit. The user data is read from the global `~/.config/oxen/user_config.toml` file. You can set your user info with the `oxen config` command.

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

What's nice about this is that once the data has been synced to the remote server, we can verify that the data is valid by computing the hashes of the files and the commit data and comparing this to the id of the commit in the database.

# Commit History

Every commit (except the first) has a list of parent commit ids. Most commits have a single parent, but in the case of a merge commit, there can be multiple parent commit ids. You can traverse the commit history by following the parent commit ids until you hit the first commit.

You can use the `oxen log` command to print out the commit history starting with the most recent commit on the current branch.

## Next Up: Branches

Learn how commits relate to [Branches](./branches.md) in the next section.