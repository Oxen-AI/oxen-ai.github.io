# This is stuff I don't wanna commit yet but taking notes

## Xet Extension

Block level deduplication does help when you have a file where you are simply changing one row at a time. But does not help when the content is different each time.

```
git add images/daisy/
Git-Xet 0.13.2: Processing data: 32.73 MiB | 29.17 MiB/s, done.           
32.73 MiB added, stored 32.73 MiB (0.0% reduction)
```

Adding image data, doing synthetic data runs, etc does not benefit from block level deduplication.

* `git status` still spews all the files you added.
* Reconstruction of files from a block level deduplication is slow. This makes it hard to take advantage of existing file formats such as parquet and arrow for what they are good at.
    * Raw Files are easier to work with for integrations.
    * TODO: Do some benchmarking



