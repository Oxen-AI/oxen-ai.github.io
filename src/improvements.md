# Improvements

Here lies a list of improvements that could be made to Oxen. This is a living document as we document the codebase and realize... Oh yeah! That can definitely be improved.

## Remove File From History

If we simply delete the file from the versions directory, and keep it's hash in the merkle tree, this makes delete quite easy.

This gets more complicated if we start to introduce pack files, etc. Which is why deleting files from the history in git is such a pain.

## Compress the data in the .oxen/versions/files/ directories

Right now we store the data in the .oxen/versions/files/ directories in a very uncompressed format of every file itself.

This is not efficient for many files that could be compressed. 

We could compress the data using zlib like git does, as well as use something like pack files to keep track of changes. The problem with packfiles is then you need to decompress the packfile to get the individual files. If you have terrabyte scale data we don't want to have to pull down multiple versions just to unpack the current. Trading off storage space with readability and compute. Hmmm.

TODO: We want to balance the benefits of native file formats with compression and data visibility. Do some benchmarking on zlib to compress a parquet file that is already compressed.

1) Encode/Decode Speed with Large Binary Files
2) Compression ratio (disk usage)

### Benefits
* Reduces the size of the .oxen/versions/files/ directories

### Drawbacks
* Time to compress and decompress the data (especially on large files that we want to be quickly accessible on the oxen server)

## Content-Type Aware Pack Files

Git uses pack files to efficiently store changes between versions without keeping entire copies of the original data. XetHub does this as well with their block level deduplication.

Their method works well with code which are relatively small text files. I think we could have a pack file format per data type that maintains the benefits of the pack file format while also being able to leverage existing file formats.

For example: Changes to images probably modify the whole file, where changes to parquet files have different properties. Would be wild to use machine learning to learn how to compress here too...but for a later date.

## Encode databases in format that is more efficient than json

This includes everything down to the hashes.

https://dev.to/calebsander/git-internals-part-1-the-git-object-model-474m

Next, we will want to be able to convert hashes back and forth between the 40-character hexadecimal representation and the compact 20-byte representation. I'll omit the implementation details, but you can find them in the source code for this post.

