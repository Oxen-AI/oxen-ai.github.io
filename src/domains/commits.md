# Commits

If you are familiar with `git` the concept of a `commit` and `branch` should be very familiar. What you may not have done is look under the hood as to how they are stored. In Oxen, many of the concepts are similar.

A commit is a checksum or [hash value](optimizations/hashing.md) representing all the files is within a specific version. You may recognize them as a string of hexadecimal characters  (0–9 and a–f) looking something like `a72b68036af144bfe2dff0fb08a746c4`.

Run `oxen log` within your Oxen repository and you will see the initial commit.

```
commit a72b68036af144bfe2dff0fb08a746c4

Author: ox
Date:   Thursday, 09 May 2024 22:29:00 +00

    Initialized Repo 🐂
```

You will see these hashes all over the place in Oxen and can use them as pointers to get to specific versions.

# Commits as Merkle Tree Nodes

Under the hood most objects in Oxen are stored in a [Merkle Tree](https://en.wikipedia.org/wiki/Merkle_tree) data structure. At the root of each merkle tree is a commit object.

All the nodes in the tree are stored in the `.oxen/tree/nodes` directory.

```
$ tree .oxen/tree/nodes/

.oxen/tree/nodes/
├── 589
│   └── 8d0aa535709791ea84a341307fc3
│       ├── children
│       └── node
├── 88b
│   └── e33604f2ae2153443bff158c31495
│       ├── children
│       └── node
└── a72
    └── b68036af144bfe2dff0fb08a746c4
        ├── children
        └── node
```

You'll see that nodes are content addressable by their hash, and each subdirectory is a level in the merkle tree. These files are hard to inspect on their own, so we can use the `oxen node` command to inspect the individual node databases.

```
$ oxen node a72b68036af144bfe2dff0fb08a746c4

CommitNode
	hash: a72b68036af144bfe2dff0fb08a746c4
	message: adding README
	parent_ids: []
	author: oxbot
	email: oxbot@oxen.ai
	timestamp: 2024-08-19 23:06:41.525894 +00:00:00
```

Here we have a nice beautiful commit object. There is a list of parent commit ids, a message, the author, the email, and the timestamp.

To see the full tree that lies below this commit, you can use the `oxen tree` command.

```
$ oxen tree -n a72b68036af144bfe2dff0fb08a746c4
```

You'll see that the tree is printed out in a human readable format. This tree only has a single README.md file in the root directory. Trees can get much more complex, and we will dive into this more in the [Merkle Trees](merkle_trees.md) section.

```
[Commit] a72b68036af144bfe2dff0fb08a746c4 -> adding README parent_ids ""
  [Dir]  -> 5898d0aa535709791ea84a341307fc3 11 B (1 nodes) (1 files) [latest commit a72b68036af144bfe2dff0fb08a746c4]
    [VNode] 88be33604f2ae2153443bff158c31495 (1 children)
      [File] README.md -> 43744a971e29c0f56c293f855f11814 11 B [latest commit a72b68036af144bfe2dff0fb08a746c4]
```

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