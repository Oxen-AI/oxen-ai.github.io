# Files, Directories and Merkle Trees 🌲

When you create a commit within Oxen, you can think of it as a snapshot of the state of the files and directories in the repository at a particular point in time. This means each commit will need a reference to all the files and directories that are present in the repository at the time of the commit.

Let's use a small dataset as an example.

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

Of course, we are not here to build naive inefficient version control tool. Oxen is a blazing fast version control system that is designed to handle large amounts of data efficiently. Even if clearing and restoring the working directory is simple, there are many reasons it is not optimal (including wiping out untracked files 😱).

## How do we make it faster?

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
README.md         -> hash1
LICENSE           -> hash2
images/image0.jpg -> hash3
images/image1.jpg -> hash4
images/image2.jpg -> hash5
images/image3.jpg -> hash6
```

Commit B

```
README.md         -> hash1 # repeated
LICENSE           -> hash2 # repeated
images/image0.jpg -> hash3 # repeated
images/image1.jpg -> hash4 # repeated
images/image2.jpg -> hash5 # repeated
images/image3.jpg -> hash6 # repeated
images/image4.jpg -> hash7 # NEW
```

Commit C

```
README.md         -> hash1 # repeated
LICENSE           -> hash2 # repeated
images/image0.jpg -> hash3 # repeated
images/image1.jpg -> hash4 # repeated
images/image2.jpg -> hash5 # repeated
images/image3.jpg -> hash6 # repeated
images/image4.jpg -> hash7 # repeated
images/image5.jpg -> hash8 # NEW
```

...

Commit 10_000

```
README.md              -> hash1
LICENSE                -> hash2
images/image0.jpg      -> hash3
images/image1.jpg      -> hash4
images/image2.jpg      -> hash5
images/image3.jpg      -> hash6
images/image4.jpg      -> hash7
images/image5.jpg      -> hash8
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

A key observation is that we are duplicating a lot of data across commits. This will be a common pattern to look for when optimizing the storage within Oxen.

# Optimizations w/ Merkle Trees

Adding one file should not require to you copy the entire key-value database. We need some sort of data structure that can efficiently store the file paths and hashes without duplicating too much data across commits.

Enter [Merkle Trees](https://en.wikipedia.org/wiki/Merkle_tree) 🌲.

Files and directories are already organized in a tree like fashion, so a Merkle Tree is a more natural fit for storing and traversing the file structure to begin with. The Oxen Merkle Tree implementation also make it so when we add additional data, we only need to copy subtrees instead of copying the entire database for each commit.

What does a Merkle Tree within Oxen look like?

![Commit A](/images/merkle_tree/commit_a.png)

At the root node of the Merkle tree is a commit hash. This is the identifier you know and love which you can reference any commit in the system by. 

The root commit hash represents the content of all the data below it, including the files contained in the `images/` directory as well as the files directly in the `.` root directory (README.md, LICENSE, etc). Additionally all the files within a directory get sectioned off into VNodes. We will return to the importance of VNodes in a bit.

At each level of the tree we see the contents of all the files hashed within that directory, and bucketed into VNodes.

## Adding a File

To see what happens when we add a new file to our repository, let's revisit our previous example of adding images to the `images/` directory. Say we have 8 images in our `images/` directory and we want to add a new image (9.jpg).

The first thing we have to do is find which VNode bucket it falls into (more on this later). Then we can recompute the hash of this subtree, and recursively update the hashes above it until we get to the root node. 

In this case we make four total updates to the tree, highlighted in green. 

1. Add the contents of the new image to our `.oxen/versions/` directory
2. Find the VNode it belongs to, and deep copy it to a new VNode with a new hash
3. Update the VNode hash of the `images/` parent directory
4. Update the root node hash

![Commit B](/images/merkle_tree/commit_b.png)

The Merkle Tree nodes are all global to the repository, and can get re-used and shared between commits. Instead of copying the entire database to our new commit, only copy the subtrees that changed. On adding a file, we only need to update a single VNode and copy it's contents. This is a much faster operation than copying every file within our databases.

For another example, let's see what happens when we update the `README.md` file.

![Commit C](/images/merkle_tree/commit_c.png)

This time, we only need to update the VNode that contains the `README.md` file and it's parent in the root node.

## What is this "VNode"?

One of the goals of Oxen is to be able to scale to directories with an arbitrary number of files. Imagine for a second that you have a directory of 100k or 1 million images. Storing all of these values directly at the directory level node would be inefficient. Every time you commit a single image to the directory, you would need to copy all the pointers and recompute the hash for the entire directory.

For example imagine we had no VNodes at the directory level.

![No VNode](/images/merkle_tree/no_vnode.png)

If we want to add a single file, we would have to copy all the pointers and recompute the hash for the entire directory.

![Add File](/images/merkle_tree/no_vnode_add_file.png)

VNode's add an intermediate bucket we can add files to so that we only have to copy a subset of pointers. Which VNode a file belongs to is computed from the hash of file path name itself. This way files get evenly distributed into buckets within the tree.

![With VNode](/images/merkle_tree/images_w_vnode.png)

You'll notice two parts to the VNode. The first is first two letters (`AB`) of the hash of the file path name, and the second is the hash of the VNode contents (`#DFEGA72`). To add an image, now we only need to find the bucket (based on it's file path), compute it's new hash, and make a copy of the items of the VNode database for it's new hash.

![With VNode Add File](/images/merkle_tree/images_w_vnode_add_file.png)

To drive this home, let's go back to our example directory with 10,000 images with the naive implementation from before. Remember 4 additions to the images directory after it contained 10,000 node resulted in 40,006 values in our database. Say our bucket size for VNodes is 10,000/256 ~= 40. This means on average we are copying 40 values with each commit. This will result in 10,160 total values in our DB instead of 40,006.


## File Chunk Deduplication

While the Merkle Tree optimizations we've talked about so far make adding and committing snapshots of the directory structure to Oxen snappy, there are even more optimizations we can do at the file level itself. 

Sometimes Oxen 🐂 can get a little chunky and need to slim down 😳.

Remember, so far each time you make a commit, we make an *entire copy* of the file contents itself and put it into the `.oxen/versions` directory under it's hash. Imagine you are editing and committing a CSV file one row at a time, and making a commit with each change. This results in a lot of duplicated data. We not only want Oxen to be efficient at storing many files, but also efficient at storing large files such as data frames of parquet, jsonl, csv, etc.

TODO: Image

To combat this, Oxen uses a technique called file chunk deduplication. Instead of storing the entire file in the `.oxen/versions` directory, if Oxen detects that a file is being modified over and over again, it chunks up the file into "chunks" and stores these chunks in the `.oxen/chunks` directory and then stores a reference to the chunks in the Merkle Tree.

TODO: Example of disk usage with and w/o file chunk dedup


## Benefits of the Merkle Tree

To summarize, there are a few nice properties of a Merkle Tree as our data structure.

1. When we add, remove or change a file, we only need to update the subtree that contains that file. This means the storage grows logarithmically with the number of files in the repository instead of linearly.

2. To recompute the root hash of a commit, we only need to hash the file paths and the hashes of the files that have changed. This means we can efficiently verify the integrity of the data by recomputing subtrees.

3. We can use it to understand the small diff of the data that needs to be transferred over the network when syncing repositories.
