# Commits

If you are familiar with `git` the concept of a `commit` and `branch` should be very familiar. What you may not have done is look under the hood as to how they are stored. In Oxen, many of the concepts are similar.

A commit is a checksum or [hash value](optimizations/hashing.md) representing all the files is within a specific version. You may recognize them as a string of hexidecimal characters  (0–9 and a–f) looking something like `4b27c4b16fb736a4`.

Run `oxen log` within your Oxen repository and you will see the initial commit.

```
commit 4b27c4b16fb736a4

Author: ox
Date:   Thursday, 09 May 2024 22:29:00 +00

    Initialized Repo 🐂
```

You will see these hashes all over the place in Oxen and can use them as pointers to get to specific versions.

