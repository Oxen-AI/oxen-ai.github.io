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
$ tree .oxen/versions/

.oxen/versions/
├── 43
│   └── 94f02b679bcf0114b1fb631c250d0a
│       └── data
├── 58
│   └── 8b7f5296c1a6041d350d1f6be41b3
│       └── data
├── 64
│   └── e1a1512c6d5b1b6dcf2122326370f1
│       └── data
├── 74
│   └── bfd17b6b7c9b183878a26e1e62a30e
│       └── data
├── 7c
│   └── 42afd26e73b8bfbc798288f1def1ed
│       └── data
├── c8
│   └── 2d11a1e1223598d930454eecfab6ea
│       └── data
└── dc
    └── 92962a4b05f5453718783fe3fc4b10
        └── data

15 directories, 7 files
```

Each file is accessible by its hash and original extension the file was stored with. For example, the hash of `images/image0.jpg` is `74bfd17b6b7c9b183878a26e1e62a30e` and it's extension is `jpg`, so the original contents can be found at `.oxen/versions/74/bfd17b6b7c9b183878a26e1e62a30e/data`.

To find the hash and extension of any file in a commit, you can use the `oxen info` command.

```
oxen info images/image0.jpg
74bfd17b6b7c9b183878a26e1e62a30e	13030	image	image/jpeg	jpg	12099a4ca3b15c36
```

The CAFS makes it easy to fetch the file data for a given commit, but we need some sort of database that lists the original file names and paths. This way when switching between commits we can efficiently restore the files that have been added/changed/removed.

# Switching Between Versions

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

## Printing the Merkle Tree

To bring these concepts to life, let's create a repo of many images and use the `oxen tree` command to print the Merkle Tree. In the [Oxen-AI/Oxen](https://github.com/Oxen-AI/Oxen) repo we have a [script](https://github.com/Oxen-AI/Oxen/blob/feat/refactor-merkle-tree/benchmark/generate_image_repo_parallel.py) that will create a directory with an arbitrary number images and add them to the `images/` directory.

When developing and testing Oxen, this script is handy to generate synthetic datasets to push the performance of the system. For example you could create a dataset of 1,000,000 images and see how long it takes to add and commit the changes.

```bash
# WARNING: This will create a directory with 1,000,000 images and take a while to run
$ python benchmark/generate_image_repo_parallel.py --output_dir ~/Data/1m_images --num_images 1000000 --num_dirs 2 --image_size 64 64
```

For this example we will stick to a smaller dataset of 20 images. It will be easier to visualize the Merkle Tree.

```bash
# 😌 This will create a much smaller dataset of 20 images
$ python benchmark/generate_image_repo_parallel.py --output_dir ~/Data/20_images --num_images 20 --num_dirs 1 --image_size 64 64
```

After the dataset is created, go ahead and create and initialize an oxen repository.

```bash
$ cd ~/Data/20_images
$ oxen init
```

Before we add and commit the files, we are going to make a quick tweak to the configuration to use a smaller VNode bucket size. The default size is 10,000, but we are going to set it to 5 to make it easier to see the tree updates in this toy example.

Edit the `.oxen/config.toml` file to set the `vnode_size` to 5.

```bash
$ cat .oxen/config.toml
remotes = []
min_version = "0.19.0"
vnode_size = 5
```

Now add and commit the files.

```bash
$ oxen add .
$ oxen commit -m "adding all data"
```

Then we can use the `oxen tree` command to print the entire Merkle Tree.

```bash
$ oxen tree

[Commit] 74aca3bd3a054a1b6942d7147acc2bf6 "adding all data" -> oxbot oxbot@oxen.ai parent_ids ""
  [Dir] 1757ac6ba211ae1e4358d8c44680bea5 "/" (249.3 KB) (commit 74aca3bd3a054a1b6942d7147acc2bf6)  (1 children)
    [VNode] 38816acec756eafbe48f6a5cba657674  (3 children)
      [File] 5d7a6a26b0b17ad87c3f36371a9bd93e "README.md" (94 B) (commit 74aca3bd3a054a1b6942d7147acc2bf6)
      [Dir] 45b852358f6cdd11c91954308438a755 "images/" (248.4 KB) (commit 74aca3bd3a054a1b6942d7147acc2bf6)  (1 children)
        [VNode] 51d73f367fcbc4f11228ff2e56fba5d3  (1 children)
          [Dir] b5476c6d51912a0cc92c42731f911daa "split_0/" (248.4 KB) (commit 74aca3bd3a054a1b6942d7147acc2bf6)  (4 children)
            [VNode] b68405755cb1df9013179d9a4335ca40  (4 children)
              [File] 972e687faf79cdecb664ebc209697c14 "noise_image_11.png" (12.4 KB) (commit 74aca3bd3a054a1b6942d7147acc2bf6)
              [File] 77c021dee25a830e685732af12e323c "noise_image_16.png" (12.4 KB) (commit 74aca3bd3a054a1b6942d7147acc2bf6)
              [File] 8ce4938cda72499ffc23765164fd509c "noise_image_5.png" (12.4 KB) (commit 74aca3bd3a054a1b6942d7147acc2bf6)
              [File] 46dcd32fb5291b4ab6d086de436afc80 "noise_image_6.png" (12.4 KB) (commit 74aca3bd3a054a1b6942d7147acc2bf6)
            [VNode] 1076ef1f91029e450bf032155babad3a  (6 children)
              [File] c8ebf33631c8a32871d456eaf72ca1c1 "noise_image_10.png" (12.4 KB) (commit 74aca3bd3a054a1b6942d7147acc2bf6)
              [File] b09dfdbac89ab19829f936a6a06e1195 "noise_image_13.png" (12.4 KB) (commit 74aca3bd3a054a1b6942d7147acc2bf6)
              [File] 5d99153957263ffa0fed8f8a9ffcd0f1 "noise_image_2.png" (12.4 KB) (commit 74aca3bd3a054a1b6942d7147acc2bf6)
              [File] eaf4c6ec29b1d45c708022f5ae1c0381 "noise_image_3.png" (12.4 KB) (commit 74aca3bd3a054a1b6942d7147acc2bf6)
              [File] a7cc0395867948f39c294b0c4a5191e9 "noise_image_7.png" (12.4 KB) (commit 74aca3bd3a054a1b6942d7147acc2bf6)
              [File] 6dcf57a3d176dc6250fd388d500b5531 "noise_image_8.png" (12.4 KB) (commit 74aca3bd3a054a1b6942d7147acc2bf6)
            [VNode] 21a5b3f9b84f58acdcbc94f37b814891  (6 children)
              [File] 15ae7d77e0b982b09be03e441c9b4482 "noise_image_0.png" (12.4 KB) (commit 74aca3bd3a054a1b6942d7147acc2bf6)
              [File] 297a0d822aa0a96b8718878d72dc3f72 "noise_image_12.png" (12.4 KB) (commit 74aca3bd3a054a1b6942d7147acc2bf6)
              [File] d84a22565ab4043194d309e1f2e8c8a2 "noise_image_14.png" (12.4 KB) (commit 74aca3bd3a054a1b6942d7147acc2bf6)
              [File] 83bb0071a97c4e1fe0ff4e15f0f2e08e "noise_image_15.png" (12.4 KB) (commit 74aca3bd3a054a1b6942d7147acc2bf6)
              [File] e61e34301aa386993a8d2fa3fb01026 "noise_image_19.png" (12.4 KB) (commit 74aca3bd3a054a1b6942d7147acc2bf6)
              [File] 38d2ac88aec48eb0ff824cb5abce113e "noise_image_4.png" (12.4 KB) (commit 74aca3bd3a054a1b6942d7147acc2bf6)
            [VNode] 8bf5b6bd09d16fc21570047b3c9f2fa  (4 children)
              [File] 252e277e354f5f23c8263b8219769037 "noise_image_1.png" (12.4 KB) (commit 74aca3bd3a054a1b6942d7147acc2bf6)
              [File] 5a884a34616843a937e6f5827c2be2ef "noise_image_17.png" (12.4 KB) (commit 74aca3bd3a054a1b6942d7147acc2bf6)
              [File] 68c7d34745eea11ec7dfd7813ee3d167 "noise_image_18.png" (12.4 KB) (commit 74aca3bd3a054a1b6942d7147acc2bf6)
              [File] 934cb1d0118b7ed5870a62a9a808e3ab "noise_image_9.png" (12.4 KB) (commit 74aca3bd3a054a1b6942d7147acc2bf6)
      [File] 251bf004b024bc93ccd989534dfdc683 "images.csv" (764 B) (commit 74aca3bd3a054a1b6942d7147acc2bf6)
```

The first thing you'll notice is that each VNode is not guaranteed to have 5 children. This is because we first instantiate the number of VNodes given the VNode size, and then we fill them with the files. The hashing algorithm will then distribute the files across the VNodes given the `file hash % number of VNodes` and some VNodes will end up with more children than others. On average a good hashing algorithm will distribute the files evenly across the VNodes.

Remember - VNodes exist in order to help us update smaller subtrees when we add, remove or change a file. To see this in action, let's add a new image to the `images/split_0` directory by converting a png to jpeg.

```
ffmpeg -i images/split_0/noise_image_11.png images/split_0/noise_image_11.jpg
oxen add images/split_0/noise_image_11.jpg
oxen commit -m "adding images/split_0/noise_image_11.jpg"
```

TODO: Show the tree before and after the commit.

## Benefits of the Merkle Tree

Hopefully if you are new to Merkle Trees, this should give you a good intuition for how they work in practice. There are a few nice properties of a Merkle Tree as our data structure.

1. When we add, remove or change a file, we only need to update the subtree that contains that file. This means the storage grows logarithmically with the number of files in the repository instead of linearly.

2. To recompute the root hash of a commit, we only need to hash the file paths and the hashes of the files that have changed. This means we can efficiently verify the integrity of the data by recomputing subtrees instead of the whole tree.

3. We can use it to understand the small chunks of the data that changes when transferring over the network when syncing repositories.

4. Since each subtree is also a merkle tree, this will allow us to clone small subtrees, make changes, and push them back up to the parent. This can be powerful when you only want to update the README for example but have a directory of images you are not planning on changing.

