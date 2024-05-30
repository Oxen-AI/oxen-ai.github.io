# Branches

Branches are a key feature of many VCS systems. They allow users to work in parallel without making changes that step on each other's toes.

The branching model in Oxen is inspired by `git` meaning branches are lightweight and quick to create. When creating a branch, we are never copying any of the raw datasets in the repository. Under the hood, a branch is really just a named reference to a commit. Creating a new branch simply creates a new named reference.

```rust
pub struct Branch {
    pub name: String,
    pub commit_id: String,
}
```

When a repository is first initialized with `oxen init`, a single commit is made and a branch is created called `main` that points to it.

# Refs

To see how this works in practice, let's look at how branches are stored on disk. All of the branches within a repository are stored in a key-value rocksdb database. This database can be found in the `.oxen/refs` directory.

Let's inspect this database with our `oxen db list` command.

```bash
$ oxen db list .oxen/refs

main	c719c887cc250784
```

This shows us that there is a single branch, `main`, that points to the commit id `c719c887cc250784`.

If we create a new branch, say `foo`, it will also be stored in the database with the same commit id as the current branch you are on.

```bash
$ oxen checkout -b foo
$ oxen db list .oxen/refs

main	c719c887cc250784
foo	c719c887cc250784
```

To see the list of current branches as well as which one you currently have checked out, you can use the `oxen branch` command.

```bash
$ oxen branch

* foo
  main
```

The `*` indicates the `foo` branch is currently checked out. The way we store the current branch is by creating a `HEAD` file in the `.oxen` directory.

This file contains the name of the branch or commit id that is currently checked out.

```bash
$ cat .oxen/HEAD

foo
```

Let's make a commit and see how the branches stored on disk change.

```bash
$ echo "foo" > foo.txt
$ oxen add foo.txt
$ oxen commit -m "foo commit"

Committing with message: foo commit
Commit 9ef4176b1b4422a7 done.
```

We now have a new commit id `9ef4176b1b4422a7`. If we look at the `refs` database, we can see that the `foo` branch has been updated to point to the new commit id.

```bash
$ oxen db list .oxen/refs

foo	9ef4176b1b4422a7
main	c719c887cc250784
```

If we look at `oxen log` we will see that the `foo` branch is now the most recent commit.

```bash
commit 9ef4176b1b4422a7

Author: Ox Bot
Date:   Thursday, 30 May 2024 04:04:53 +00

    foo commit

commit c719c887cc250784

Author: Ox Bot
Date:   Tuesday, 28 May 2024 03:03:49 +00

    adding questions.jsonl
```

You can checkout a specific commit by using the `oxen checkout` command with the commit id.

```bash
$ oxen checkout c719c887cc250784
```

This will update the `HEAD` file to point to the commit id instead of the branch name. 

```bash
$ cat .oxen/HEAD

c719c887cc250784
```

You will notice that our `foo.txt` file is no longer present in the working directory. If you perform a `oxen status` you will see that we are now in a "detached HEAD" state. This means that we are no longer on a branch and are instead on an individual commit.

Don't worry, the file `foo.txt` is still alive and well in the `.oxen/versions` directory, and can be restored by checking out the `foo` branch again.

```bash
$ oxen checkout foo
```

That's it! The relationship between branches, commits, and the HEAD commit is really that simple. Branches are just a named reference to a commit id that make it easier to find a particular chain of commits.

You can progress a branch as many commits as you want without affecting the main branch. When you are ready to merge your branch into the main branch, you can use the `oxen merge` command which will be covered later.

# Next Up: Files & Directories

Now that you know the basic data structures for branches and commits, let's dive into how files and directories are stored in Oxen.

Next Up: [Files & Directories](/domains/objects.md)