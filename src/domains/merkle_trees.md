# Files, Directories and Merkle Trees 🌲

When you create a commit within Oxen, you can think of it as a snapshot of the state of the files and directories in the repository at a particular point in time. This means each commit will need a reference to all the files and directories that are present in the repository at that point in time.

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

Of course, we are not here to build naive inefficient version control tool. Oxen is a blazing fast version control system that is designed to handle large amounts of data efficiently. Even if clearing and restoring the working directory is simple, there are many reasons it is not optimal (including wiping out untracked files).

## Data Duplication 😥

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
README.md         -> hash1 # repeated 1 time
LICENSE           -> hash2 # repeated 1 time
images/image0.jpg -> hash3 # repeated 1 time
images/image1.jpg -> hash4 # repeated 1 time
images/image2.jpg -> hash5 # repeated 1 time
images/image3.jpg -> hash6 # repeated 1 time
images/image4.jpg -> hash7 # NEW
```

Commit C

```
README.md         -> hash1 # repeated 2 times
LICENSE           -> hash2 # repeated 2 times
images/image0.jpg -> hash3 # repeated 2 times
images/image1.jpg -> hash4 # repeated 2 times
images/image2.jpg -> hash5 # repeated 2 times
images/image3.jpg -> hash6 # repeated 2 times
images/image4.jpg -> hash7 # repeated 1 time
images/image5.jpg -> hash8 # NEW
```

...

Commit 10_000

```
README.md              -> hash1 # repeated N times
LICENSE                -> hash2 # repeated N times
images/image0.jpg      -> hash3 # repeated N times
images/image1.jpg      -> hash4 # repeated N times
images/image2.jpg      -> hash5 # repeated N times
images/image3.jpg      -> hash6 # repeated N times
images/image4.jpg      -> hash7 # repeated N times
images/image5.jpg      -> hash8 # repeated N times
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

Adding one file should not require to you copy the duplicate all the file paths and hashes from the previous commit. We need some sort of data structure that can efficiently store the file paths and hashes without duplicating too much data across commits.

Enter [Merkle Trees](https://en.wikipedia.org/wiki/Merkle_tree) 🌲.

Files and directories are already organized in a tree like fashion, so a Merkle Tree is a more natural fit for storing and traversing the file structure to begin with. The Oxen Merkle Tree implementation also makes it so when we add additional data, we only need to copy subtrees instead of copying the entire database for each commit.

What does a Merkle Tree within Oxen look like?

![Commit A](/images/merkle_tree/commit_a.png)

At the root node of the Merkle tree is a commit hash. This is the identifier you know and love which you can reference any commit in the system by.

Merkle trees hashes are constructed by recursively combining hashes of the nodes below it. Each file and directory in our tree gets it's own hash based on it's content. There are some intermediate nodes that we will dive into detail later, but for now think of a directory hash as the combination of the hashes of all the files and directories within it.

## Printing a Merkle Tree

Oxen has a convenient command that prints out the Merkle Tree for a given commit if you need to debug or simply are curious.

```
oxen tree
```

TODO: add terminal outputs

## Sharing Nodes Between Commits

The trick we are going to use to reduce storage is to share tree nodes between commits. This means only updating and copying sub-trees that have changed.

For example, say we have a repository with a `README.md` and `LICENSE` and the root, plus 8 images in the `images/` directory. If we only modify the `README.md` between Commit A and Commit B, there is no need to recompute hashes and copy pointers to all the files in the `images/` directory.

![Commit README](/images/merkle_tree/commit_modify_readme.png)

The node for `images/` is shared between Commit A and Commit B, but we make a new copy of the node for the `README.md` and `LICENSE` for Commit B.

As another example, what if we want to add image `9.jpg` to the `images/` directory?

![Commit B](/images/merkle_tree/commit_b.png)

In this case, since there are potentially many images in the directory, there is one more level of bucketing. We refer to these intermediate buckets as "VNodes" (more on this later). The first thing we have to do is find which VNode bucket it falls into, then we can recompute the hash of this subtree. We then recursively update the hashes above it until we get to the root node. VNodes are kept relatively small to limit the number of copies of these pointers and their hashes.

When adding the image in our subdirectory, we perform one deep copy of the leaf VNode database, add the image pointer to the list of pointers. Then we recursively deep copy and update the parent nodes until we get to the root node. These copies are kept small since we limit the number of pointers per node.

## Storage on Disk

All tree nodes are stored on disk in the directory `.oxen/tree`. Each node gets it's own directory and database to make it quite fast to read the individual nodes as well as to update and write them.

TODO: Show storage

## Why use VNodes?

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

# File Chunk Deduplication

The Merkle Tree optimizations we've talked about so far make adding and committing snapshots of the directory structure to Oxen snappy. The remainder of the storage cost is at the individual leaf nodes of the tree. We want Oxen to be efficient at storing many files as well as efficient at storing large files. In our context these large files may be data frames of [parquet](https://parquet.apache.org/), [arrow](https://arrow.apache.org/), [jsonl](https://jsonlines.org/), [csv](https://en.wikipedia.org/wiki/Comma-separated_values), etc.

To visualize how much storage space individual nodes take, let's look at a large CSV in the `.oxen/versions` directory. The pointers and hashes within the tree itself are relatively small, but the file contents themselves are large.

![Large CSV](/images/merkle_tree/large_csv.png)

Remember, so far each time you make a commit, we make an **entire copy** of the file contents itself and put it into the `.oxen/versions` directory under it's hash.

![Large CSV](/images/merkle_tree/large_csv_version.png)

This can be a problem if we keep updating the same file over over and over again. Five small changes to a 1GB file will result in 5GB of storage.

Think back to the key insight that we made earlier about duplicating data between versions of our tree. The same thing applies to large files. For example - what if you are editing and committing a CSV file one row at a time.

![Prompts CSV](/images/merkle_tree/prompts_csv.png)

This results in a lot of duplicated data. In fact rows 0-1,000,000 are all the same between Version A and Version B.

To combat this, Oxen uses a technique called file chunk deduplication. Instead of duplicating the entire raw file in the `.oxen/versions` directory, we can break the file into "chunks" then store these chunks in a content addressable manner. 

![Chunks](/images/merkle_tree/chunk_a.png)

Then if you update any rows in the file, we create a new chunk for that section and update the leaf nodes of the tree.

![Chunks](/images/merkle_tree/chunks_a_b.png)

So the original tree looks like this.

![Chunks](/images/merkle_tree/tree_chunks.png)

Then when the row is added, all we have to do is update the final chunk and propagate the changes up the tree.

![Update Chunk](/images/merkle_tree/tree_chunks_update.png)

What's great about this, is now we also only need to sync the chunks that have changed over the network and not the entire file. Chunks themselves are ~16kb in size, so updating a single value in a 1GB file will only sync this small portion of the file.

When swapping between versions we simply reconstruct the chunks associated with each file and write them to disk. This means we need one more small db that stores the mapping from chunk_idx -> chunk_hash. We can store this in the `.oxen/versions` directory instead of the file contents, saving space!

## Benefits of the Merkle Tree

To summarize, there are a few nice properties of a Merkle Tree as our data structure.

1. When we add, remove or change a file, we only need to update the subtree that contains that file. This means the storage grows logarithmically with the number of files in the repository instead of linearly.

2. To recompute the root hash of a commit, we only need to hash the file paths and the hashes of the files that have changed. This means we can efficiently verify the integrity of the data by recomputing subtrees.

3. We can use it to understand the small diff of the data that needs to be transferred over the network when syncing repositories.

