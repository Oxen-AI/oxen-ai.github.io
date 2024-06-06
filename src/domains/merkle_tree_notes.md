
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

## VNodes

TODO: Describe the importance of vnodes.

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