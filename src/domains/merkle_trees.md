# Files, Directories and Merkle Trees 🌲

When you create a commit within Oxen, you can think of it as a snapshot of the state of the files and directories in the repository at a particular point in time. This means each commit will need a reference to all the files and directories that are present in the repository at the time of the commit.

Let's use an example file structure to learn the ins and outs of Oxen's file system.

Commit A

```
README.md
LICENSE
images/
  image0.jpg
  image1.jpg
  image2.jpg
  image3.jpg
  image4.jpg
```

This is a simple file structure with a README.md at the top level and a sub-directory of images. Start by initializing a new repository, then adding and committing the files.

```bash
oxen init
oxen add README.md
oxen add images/
oxen commit -m "adding data"
```

On commit we save off all the hashes of the file contents and save the data into a Content Addressable File System (CAFS) within the `.oxen/versions` directory. This makes it so we don't duplicate the same file data across commits. 

```bash
$ tree .oxen/versions/files/

.oxen/versions/files/
├── 43
│   └── 94f02b679bcf0114b1fb631c250d0a
│       └── data.jpg
├── 58
│   └── 8b7f5296c1a6041d350d1f6be41b3
│       └── data.jpg
├── 64
│   └── e1a1512c6d5b1b6dcf2122326370f1
│       └── data.md
├── 74
│   └── bfd17b6b7c9b183878a26e1e62a30e
│       └── data.jpg
├── 7c
│   └── 42afd26e73b8bfbc798288f1def1ed
│       └── data
├── c8
│   └── 2d11a1e1223598d930454eecfab6ea
│       └── data.jpg
└── dc
    └── 92962a4b05f5453718783fe3fc4b10
        └── data.jpg

15 directories, 7 files
```

Each file is accessible by its hash and original extension the file was stored with. For example, the hash of `images/image0.jpg` is `74bfd17b6b7c9b183878a26e1e62a30e` and it's extension is `jpg`, so the original contents can be found at `.oxen/versions/files/74/bfd17b6b7c9b183878a26e1e62a30e/data.jpg`.

To find the hash and extension of any file in a commit, you can use the `oxen info` command.

```
oxen info images/image0.jpg
74bfd17b6b7c9b183878a26e1e62a30e	13030	image	image/jpeg	jpg	12099a4ca3b15c36
```

The CAFS makes it easy to fetch the file data for a given commit, but we need some sort of database that lists the original file names and paths. This way when switching between commits we can efficiently restore the files that have been added/changed/removed.

# Naive Implementation

The simplest solution would be to have a key-value database for every commit that listed the file paths and pointed to their hashes and extensions.

Commit A

```
README.md -> {"hash": "64e1a1512c6d5b1b6dcf2122326370f1", "extension": ".md"}
LICENSE -> {"hash": "7c42afd26e73b8bfbc798288f1def1ed", "extension": ""}
images/image1.jpg -> {"hash": "74bfd17b6b7c9b183878a26e1e62a30e", "extension": ".jpg"}
images/image2.jpg -> {"hash": "dc92962a4b05f5453718783fe3fc4b10", "extension": ".jpg"}
images/image3.jpg -> {"hash": "588b7f5296c1a6041d350d1f6be41b3", "extension": ".jpg"}
images/image4.jpg -> {"hash": "c82d11a1e1223598d930454eecfab6ea", "extension": ".jpg"}
images/image5.jpg -> {"hash": "4394f02b679bcf0114b1fb631c250d0a", "extension": ".jpg"}
```

We could store this in a rocksdb database in `.oxen/history/{commit_hash}/files`. The keys would be the file paths and the values would be the hashes and extensions. Then when swapping between commits all we would have to do is clear the current working directory and re-construct all the files from the respective commit database!

Psuedo Code:

```bash
set commit_hash 1d278f841510b8e7
rm -rf working_dir
for dir, hash, ext in (oxen db list .oxen/versions/files/$commit_hash/) ; 
  mkdir -p working_dir/$dir ;
  cp .oxen/versions/files/$commit_hash/$hash/data$ext working_dir/$dir/ ;
end
```

Version control complete. Let's call it a day and go relax on the beach 😎 🏝️.

Of course, we are not here to build naive inefficient version control tool. Oxen is a blazing fast version control system that is designed to handle large amounts of data efficiently. Even if clearing and restoring the working directory is simple, there are many reasons it is not optimal. 

## Why Is This Inefficient?

To see why this naive approach is sub-optimal, imagine we are collecting image training data for a computer vision system. We put Oxen in a loop adding one new image at a time to the `images/` directory. Each time we add an image we commit the changes.

```fish
for i in (seq 100) ; 
  # imaginary data collection pipeline
  cp /path/to/images/image$i.jpg images/image$i.jpg ;

  # oxen add and commit
  oxen add images/image$i.jpg ;
  oxen commit -m "adding image$i" ;
end
```

If we had gone the naive route, this would balloon in redundancy even with just our list of pointers to hashes. Each database list repeats the same file paths and the hashes over and over again.

Commit A

```
README.md -> hash1
LICENSE -> hash2
images/image0.jpg -> hash3
images/image1.jpg -> hash4
images/image2.jpg -> hash5
images/image3.jpg -> hash6
```

Commit B

```
README.md -> hash1
LICENSE -> hash2
images/image0.jpg -> hash3
images/image1.jpg -> hash4
images/image2.jpg -> hash5
images/image3.jpg -> hash6
images/image4.jpg -> hash7
```

Commit C

```
README.md -> hash1
LICENSE -> hash2
images/image0.jpg -> hash3
images/image1.jpg -> hash4
images/image2.jpg -> hash5
images/image3.jpg -> hash6
images/image4.jpg -> hash7
images/image5.jpg -> hash8
```

...

Commit 10_000

```
README.md -> hash1
LICENSE -> hash2
images/image0.jpg -> hash3
images/image1.jpg -> hash4
images/image2.jpg -> hash5
images/image3.jpg -> hash6
images/image4.jpg -> hash7
images/image5.jpg -> hash8
...
images/image10_000.jpg -> hash10_000
```

Do the math once we get to a dataset of 10,000 images. Each commit duplicates 10,000+1 values. 10,000 + 10,001 + 10,002 + 10,003 = 40,006 values in our collective databases.

```
.oxen/history/COMMIT_A/files -> 10,000 values
.oxen/history/COMMIT_B/files -> 10,001 values
.oxen/history/COMMIT_C/files -> 10,002 values
.oxen/history/COMMIT_D/files -> 10,003 values

Total Values: 40,006
```

# Optimizations w/ Merkle Trees

Clearly there is a lot of shared data between the commits above... Adding one file should not require to you copy the entire key-value database. We need some sort of data structure that can efficiently store the file paths and hashes without duplicating too much data across commits.

Enter [Merkle Trees](https://en.wikipedia.org/wiki/Merkle_tree) 🌲.

Files and directories are already organized in a tree like fashion, so a merkle tree is a more natural fit for storing and traversing the file structure to begin with. They also make it so when we add additional data, we only need to copy subtrees instead of copying the entire database for each commit.

What does a Merkle Tree within Oxen look like?

![Commit A](/images/merkle_tree/commit_a.png)

At the root node of the Merkle tree is a commit hash. This is the identifier you know and love which you can reference any commit in the system by. 

The root commit hash represents the content of all the data below it, including the files contained in the `images/` directory as well as the files directly in the `.` root directory (README.md, LICENSE, etc). Additionally all the files within a directory get sectioned off into VNodes. We will return to the importance of VNodes in a bit.

At each level of the tree we see the contents of all the files hashed within that directory, and bucketed into VNodes.

There are a few nice properties of storing everything in a Merkle Tree.

1. When we add, remove or change a file, we only need to update the subtree that contains that file. This means the storage grows logarithmically with the number of files in the repository instead of linearly.

2. To recompute the root hash of a commit, we only need to hash the file paths and the hashes of the files that have changed. This means we can efficiently verify the integrity of the data by recomputing subtrees.

3. We can use it to understand the small diff of the data that needs to be transferred over the network when syncing repositories.

TODO: Articulate these value props more clearly ^

## Adding a File

To see what happens when we add a new file to our repository, let's revisit our previous example of adding images to the `images/` directory. Say we have 8 images in our `images/` directory and we want to add a new image (9.jpg).

The first thing we have to do is find which VNode bucket it falls into. Then we can recompute the hash of this subtree, and recursively update the hashes above it until we get to the root node. In this case we make four total updates to the tree, highlighted in green. 

1. Add the contents of the new image
2. Update it's VNode hash
3. Update the VNode hash of the `images/` parent directory
4. Update the root node hash

![Commit B](/images/merkle_tree/commit_b.png)

Instead of copying the entire database to our new commit, we only need to copy pointers to the nodes that have changed. Let's look at what this looks like on disk.

TODO: `oxen db list .oxen/objects/...`

## VNodes

TODO: Describe the importance of vnodes.