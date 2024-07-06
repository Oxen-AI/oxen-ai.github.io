
# Local Commit Merkle Tree Nodes

In the `.oxen/history` directory we will keep each individual commit. For example we might have two commits, `498071b3d6d21ba7` and `691398fedd80ff94`. This means we will have to sub directories.

`.oxen/history/498071b3d6d21ba7/`

`.oxen/history/691398fedd80ff94/`

Each one of these represents the entire state of the repo at that point in time. Within each of these commit directories we will have a subdirectory called `tree` that represents the merkle tree for that commit.

`.oxen/history/498071b3d6d21ba7/tree/`

It is a rocksdb of `directory` -> `merkle_tree_hash` at that level. There may be a few levels of directories. Let's take this repository of Flowers as an example.

```
.
├── LICENSE
├── README.md
├── images
│   ├── daisy
│   ├── dandelion
│   ├── roses
│   ├── sunflowers
│   └── tulips
├── train.csv
├── test.csv
```

This would result in a database of:

```bash
$ oxen db list .oxen/history/498071b3d6d21ba7/tree/__/

.	64f2e2e90a49d4fe9f52b95a053ad3fe
images	7e15655cad8288df9c902e2d44271b94
```

Where the keys are the directory names and the values are the merkle tree hashes. There will then be sub databases if there are sub directories, such as in the `images` directory.

```bash
$ oxen db list .oxen/history/498071b3d6d21ba7/tree/images/

.	74f2e2e90a49d4fe9f52b95a053ad3fe
daisy	f1b6313dc9523b5591b393eee9a5852f
dandelion	4574d24cc30b99bac382f332b8b87cef
roses	7f8e686bab69533c8584ccf0657cf6b6
sunflowers	5574d24cc30b99bac382f332b8b87cef
tulips	f2a6313dc9523b5591b393eee9a5852f
```

Each one of these merkle tree nodes may be shared across multiple commits. This means to figure out what is inside the commit we need to look at the global `tree` database.

# Global Merkle Tree Nodes

Since Merkle Tree Nodes can be shared across multiple commits, we need a shared location to look them up. This is the `.oxen/tree` database.

Each one of the hashes in the `.oxen/history/<commit>/tree/<dir>/` sub databases will be a key in the `.oxen/tree` database. These databases are also scoped by directory, so we will have a `.oxen/tree/<dir>/` sub database.

Let's look at the root of the tree, which is the `.oxen/tree/` databases. There will be an individual database for each merkle node hash.

```bash
ls .oxen/tree/

64f2e2e90a49d4fe9f52b95a053ad3fe
7e15655cad8288df9c902e2d44271b94
```

This is every merkle tree node that has ever existed at this level of the directory.

Then we can list all the nodes that are children of this node.

```bash
$ oxen db list .oxen/tree/64f2e2e90a49d4fe9f52b95a053ad3fe

f9c5b535269cb6255eb15de5addd2788 {"VNode": { "name": "AA" }}
f968ede8f2d62be64a2f15fcf68c4cb7 {"VNode": { "name": "BB" }}
f0c1d88961b01e825fefaee45fcf095d {"VNode": { "name": "CC" }}
edd0cd8df136708b389ea2f43aad6d04 {"VNode": { "name": "DD" }}
```

Which means there are 4 children of this node. We can now go and look at the children of these nodes. Finally we are down to the files.

```bash
$ oxen db list .oxen/tree/f9c5b535269cb6255eb15de5addd2788

64e1a1512c6d5b1b6dcf2122326370f1 {"File": { "path": "README.md", "hash": "64e1a1512c6d5b1b6dcf2122326370f1","num_bytes":2484 }}
54e1a1512c6d5b1b6dcf2122326370f1 {"ChunkedFile": { "path": "data.csv", "hash": "54e1a1512c6d5b1b6dcf2122326370f1","num_bytes":2484 }}
```

If we have a File object it's a leaf node, if we have a ChunkedFile object then there are chunks to be stored.

```bash
$ oxen db list .oxen/tree/f9c5b535269cb6255eb15de5addd2788/54e1a1512c6d5b1b6dcf2122326370f1

TODO: chunks....
```

# Fixing Merkle Trees

TODO: I think this really needs a refactor/migration...We put everything at the top level files,vnodes,dirs databases. Traversing down, this is what it currently looks like:

Directories

```
$ oxen db list .oxen/objects/dirs/

35a5fb306ca8e5ba4ac151cc69cbb78a	{"Dir":{"children":[{"VNode":{"path":"19","hash":"219c082abe2f0c8667137caf4e8754f"}},{"VNode":{"path":"ee","hash":"3c30cf023725bd67959aeaaf1b84b057"}}],"hash":"35a5fb306ca8e5ba4ac151cc69cbb78a"}}
824464b6325fdd0aca84f08e9a55c4d1	{"Dir":{"children":[{"VNode":{"path":"19","hash":"219c082abe2f0c8667137caf4e8754f"}},{"VNode":{"path":"3f","hash":"80329f520fe06cfef56a800a1d1449ee"}},{"VNode":{"path":"ee","hash":"3c30cf023725bd67959aeaaf1b84b057"}}],"hash":"824464b6325fdd0aca84f08e9a55c4d1"}}
99aa06d3014798d86001c324468d497f	{"Dir":{"children":[],"hash":"99aa06d3014798d86001c324468d497f"}}
b3083b557eefe7437b486dec8e335823	{"Dir":{"children":[{"VNode":{"path":"a4","hash":"d7eae793771625acfc92a81d7ddf9263"}}],"hash":"b3083b557eefe7437b486dec8e335823"}}
b91aa7cc12af08d1028c8409a9f5ed1d	{"Dir":{"children":[{"VNode":{"path":"75","hash":"974f2209cf5bb5c4a8d3f75c244f0d21"}},{"VNode":{"path":"a7","hash":"1f2d7abd7b14a084884ca822dcd6a16"}},{"VNode":{"path":"ab","hash":"a4c458c5d6598d425e5240ed4acaa806"}},{"VNode":{"path":"ca","hash":"488bc4f4cedaf8a3df56aaa89d746477"}},{"VNode":{"path":"e6","hash":"29ec84756de3b9185011fed9ef3876fe"}}],"hash":"b91aa7cc12af08d1028c8409a9f5ed1d"}}
```

VNodes

```
$ oxen db list .oxen/objects/vnodes/

1f2d7abd7b14a084884ca822dcd6a16	{"VNode":{"children":[{"File":{"path":"images/train/image0.jpg","hash":"74bfd17b6b7c9b183878a26e1e62a30e"}}],"hash":"1f2d7abd7b14a084884ca822dcd6a16","name":"a7"}}
219c082abe2f0c8667137caf4e8754f	{"VNode":{"children":[{"File":{"path":"README.md","hash":"64e1a1512c6d5b1b6dcf2122326370f1"}}],"hash":"219c082abe2f0c8667137caf4e8754f","name":"19"}}
29ec84756de3b9185011fed9ef3876fe	{"VNode":{"children":[{"File":{"path":"images/train/image4.jpg","hash":"4394f02b679bcf0114b1fb631c250d0a"}}],"hash":"29ec84756de3b9185011fed9ef3876fe","name":"e6"}}
3c30cf023725bd67959aeaaf1b84b057	{"VNode":{"children":[{"Dir":{"path":"images","hash":"b3083b557eefe7437b486dec8e335823"}}],"hash":"3c30cf023725bd67959aeaaf1b84b057","name":"ee"}}
488bc4f4cedaf8a3df56aaa89d746477	{"VNode":{"children":[{"File":{"path":"images/train/image3.jpg","hash":"c82d11a1e1223598d930454eecfab6ea"}}],"hash":"488bc4f4cedaf8a3df56aaa89d746477","name":"ca"}}
80329f520fe06cfef56a800a1d1449ee	{"VNode":{"children":[{"File":{"path":"LICENSE","hash":"7c42afd26e73b8bfbc798288f1def1ed"}}],"hash":"80329f520fe06cfef56a800a1d1449ee","name":"3f"}}
974f2209cf5bb5c4a8d3f75c244f0d21	{"VNode":{"children":[{"File":{"path":"images/train/image1.jpg","hash":"dc92962a4b05f5453718783fe3fc4b10"}}],"hash":"974f2209cf5bb5c4a8d3f75c244f0d21","name":"75"}}
a4c458c5d6598d425e5240ed4acaa806	{"VNode":{"children":[{"File":{"path":"images/train/image2.jpg","hash":"588b7f5296c1a6041d350d1f6be41b3"}}],"hash":"a4c458c5d6598d425e5240ed4acaa806","name":"ab"}}
d7eae793771625acfc92a81d7ddf9263	{"VNode":{"children":[{"Dir":{"path":"images/train","hash":"b91aa7cc12af08d1028c8409a9f5ed1d"}}],"hash":"d7eae793771625acfc92a81d7ddf9263","name":"a4"}}
```

Files

```
$ oxen db list .oxen/objects/files/

4394f02b679bcf0114b1fb631c250d0a	{"File":{"hash":"4394f02b679bcf0114b1fb631c250d0a","num_bytes":6510,"last_modified_seconds":1717193404,"last_modified_nanoseconds":178666106}}
588b7f5296c1a6041d350d1f6be41b3	{"File":{"hash":"588b7f5296c1a6041d350d1f6be41b3","num_bytes":35350,"last_modified_seconds":1717193381,"last_modified_nanoseconds":270961618}}
64e1a1512c6d5b1b6dcf2122326370f1	{"File":{"hash":"64e1a1512c6d5b1b6dcf2122326370f1","num_bytes":5,"last_modified_seconds":1717194393,"last_modified_nanoseconds":555589712}}
74bfd17b6b7c9b183878a26e1e62a30e	{"File":{"hash":"74bfd17b6b7c9b183878a26e1e62a30e","num_bytes":13030,"last_modified_seconds":1717193321,"last_modified_nanoseconds":582627035}}
7c42afd26e73b8bfbc798288f1def1ed	{"File":{"hash":"7c42afd26e73b8bfbc798288f1def1ed","num_bytes":17,"last_modified_seconds":1717258120,"last_modified_nanoseconds":215692298}}
c82d11a1e1223598d930454eecfab6ea	{"File":{"hash":"c82d11a1e1223598d930454eecfab6ea","num_bytes":30689,"last_modified_seconds":1717193390,"last_modified_nanoseconds":517968946}}
dc92962a4b05f5453718783fe3fc4b10	{"File":{"hash":"dc92962a4b05f5453718783fe3fc4b10","num_bytes":17331,"last_modified_seconds":1717193350,"last_modified_nanoseconds":154739259}}
```

This structure makes the individual databases to be too large. Really you should only open a db per directory.

I think it should more be like:

```
# list top level dirs / vnodes
oxen db list .oxen/objects/dirs/

There are two items: images directory and AA vnode
[
  Dir {
    path: "images",
    hash: "b3083b557eefe7437b486dec8e335823",
  },
  VNode {
    path: "AA",
    hash: "a3083b557eefe7437b486dec8e335823",
  },
]

# list all vnodes / subdirs in a dir
oxen db list .oxen/objects/dirs/images/

There are two items: AA and BB vnodes
[
  VNode {
    path: "AB",
    hash: "b3083b557eefe7437b486dec8e335823",
  },
  VNode {
    path: "BB",
    hash: "a3083b557eefe7437b486dec8e335823",
  },
]
```

Then you can go look up the files in the AB vnode

```
oxen db list .oxen/objects/vnodes/AB/
```

or the BB vnode

```
oxen db list .oxen/objects/vnodes/images/BB/
```

This gets rid of file paths stored in the vnodes themselves which makes the cross platform representation of the data easier and more efficient (smaller less duplicated strings on disk). It will also make the objects database faster to open, because right now it is a bottle neck.


# RocksDB Tree Structure

Our old databases have an undesirable pattern of putting full paths within the values. For example...

```bash
oxen db list .oxen/objects/vnodes/

f0c1d88961b01e825fefaee45fcf095d	{"VNode":{"children":[{"File":{"path":"images/test/dandelion/136999986_e410a68efb_n.jpg","hash":"4bafb0f05abd3e3c77a687a2d970d070"}}],"hash":"f0c1d88961b01e825fefaee45fcf095d","name":"e5"}}
f968ede8f2d62be64a2f15fcf68c4cb7	{"VNode":{"children":[{"File":{"path":"images/test/dandelion/16159487_3a6615a565_n.jpg","hash":"7351d0369bbe690e6b3852dfd04ccb5f"}}],"hash":"f968ede8f2d62be64a2f15fcf68c4cb7","name":"9f"}}
f9c5b535269cb6255eb15de5addd2788	{"VNode":{"children":[{"Dir":{"path":"images/test/tulips","hash":"96a07ff0121d7e4dce6433098569d1bf"}}],"hash":"f9c5b535269cb6255eb15de5addd2788","name":"7c"}}
fbcea208f083ac0197abcfd62b6417c	{"VNode":{"children":[{"File":{"path":"images/test/tulips/116343334_9cb4acdc57_n.jpg","hash":"9316adb67466114e93d85a1ca01f7b8c"}}],"hash":"fbcea208f083ac0197abcfd62b6417c","name":"de"}}
```

You'll see that we have paths like `images/test/dandelion/136999986_e410a68efb_n.jpg` within the values. 

The first problem is this is not platform agnostic between windows and linux, so we have to do some platform specific logic.

The second problem is now we have 1 large database for all the vnodes. It is more more efficient to have a database per directory.

We want to refactor it so that we have a database per directory, that only lists the nodes within that directory. 

```bash
oxen db list .oxen/objects/vnodes/images/tests/tulips

fbcea208f083ac0197abcfd62b6417c	{"VNode":{"children":[{"File":{"path":"116343334_9cb4acdc57_n.jpg","hash":"9316adb67466114e93d85a1ca01f7b8c"}}],"hash":"fbcea208f083ac0197abcfd62b6417c","name":"de"}}
```

Then the path is just the file name at that location.

TODO: Remove this statement above after the refactor.

## General Design Principle

TODO: Put this in coding guidelines?

Do not put `/` in paths within the rocks dbs themselves. Instead have sub directories of rocks dbs per directory. For example, we could have a single rocksdb of all directories.

```bash
$ oxen db list .oxen/history/498071b3d6d21ba7/dirs/

.	0
code	0
images	0
images/daisy	0
images/dandelion	0
images/roses	0
images/sunflowers	0
images/tulips	0
metadata	0
```

But it is preferable to only have the sub directories at the level you are looking at.

```bash
$ oxen db list .oxen/history/498071b3d6d21ba7/dirs/

.	0
code	0
images	0
metadata	0
```

Then sub rocks dbs at each level. For example, `images` directory.

```bash
$ oxen db list .oxen/history/498071b3d6d21ba7/dirs/images/

daisy	0
dandelion	0
roses	0
sunflowers	0
tulips	0
```

This helps with cross platform representation of the data. We don't have to translate slashes from unix to windows. It also helps keep each individual rocksdb smaller. This will result in faster lookups at the individual depth you are looking at, rather than having a large database at the top level.


## Next Up

Block level de-dup, compression. With our ability to edit dfs, this is the perfect time to start looking into how to best compress data.

Edit a row and commit over and over again, what is the storage cost? Can we optimize?

Run some quick experiments with parquet files, csv, arrow, jsonl, model + lora weights, etc.

1) Add row
2) Add column
3) Update single value
4) See how many bytes overlap
5) Naive chunk and add to merkle tree
6) How fast it is to reconstruct from chunks / read into polars


# METRICS

## Nested RocksDBs:

Disk Usage

```
~/D/CelebA> du -hs .oxen/tree/
 53M	.oxen/tree/
```

Time to read

```
~/D/CelebA>  time oxen tree -c 7b495f06130f9a8 > /tmp/log.txt

________________________________________________________
Executed in    3.01 secs    fish           external
   usr time    2.69 secs    0.15 millis    2.69 secs
   sys time    0.43 secs    1.41 millis    0.43 secs
```

```
~/D/CelebA>  time oxen tree -c 7b495f06130f9a8 > /tmp/log.txt

________________________________________________________
Executed in    3.61 secs    fish           external
   usr time    2.71 secs    0.22 millis    2.71 secs
   sys time    0.45 secs    1.68 millis    0.45 secs
```

## Remove Hash from VNode

```
~/D/CelebA> du -hs .oxen/tree/
 22M	.oxen/tree/

________________________________________________________
Executed in    2.42 secs    fish           external
   usr time    2.08 secs    0.22 millis    2.08 secs
   sys time    0.38 secs    2.07 millis    0.38 secs
```

# MessagePack

```
~/D/CelebA> du -hs .oxen/tree/
 19M	.oxen/tree/
```

# 1 Million Files Benchmarks

## Million Files Many Subdirs

This is more like ImageNet

I actually think this is the more likely path. We over-fit to CelebA a bit.

TODO:
* Compute number of VNode Buckets based on number of sub files and directories. Should be N / (2^M) <= 10,000. If N is known then it's N / 10,000 = (2^M) .... M = log2(N / 10000)
  * log2(1,000,000 / 10,000)
    * 1,000,000,000 / (2^16) = 1,000,000,000 / 65,536 = 15,258
    * 1,000,000 / (2^6) = 1,000,000 / 64 = 15,625
    * 500,000 / (2^5) = 500,000 / 32 = 15,625
    * 200,000 / (2^4) = 200,000 / 16 = 12,500
* Split dir,file,schema into their own vnode types DVNode, FVNode, SVNode
  * This will make it much easier to paginate the directories and files stacked on top of each other
  * dir_a/
  * dir_b/
  * dir_c/
  * README.md

## Million Files Single Directory

Okay 1 million files in a single directory, we sped up greatly with the new architecture.
...65x speed up.

### Time To Load Tree Page

Old Merkle Tree:

```
~> oxen tree -c 48c579ed89dcfa6d -d 1 --old
Time to load tree: 197.668902333s
```

New Merkle Tree:

```
~> oxen tree -c 48c579ed89dcfa6d -d 1
Time to load tree: 3.648578459s
```

