# Optimizations

There are many optimizations that make Oxen go brrr. From the core merkle tree structure, to hashing protocol, to networking, to interacting with remote datasets. Oxen is meant to make it feel like you have terabytes of data at your fingertips whatever machine you are on.

# ~TLDRs~

We often get asked: "What makes Oxen different from other VCS?"

Without diving into the gnitty gritty details, here are some highlights. If you want to go deeper, don't worry, we also dive deep into the implementation details of each throughout the book.

## Merkle Tree

* Downloading Sub Trees
* Per Folder Sub Trees
* Block Level Dedup
* Only download latest
    * When you get to TB scale data, you do not want to have to pull down data from previous commits to compute the current tree.
* Push / Pull Bottle Neck
* Objects
    * Trees
    * VNodes
    * Blobs
    * Schemas

## Hashing

* xxHash
* pure hashing throughput
* non-cryptographic hashing fn

## Data Frames and Schemas are First Class Citizens

Other VCS systems are optimized for text files and code. In the case of datasets, we often deal with data frames which have other properties such as schema that we want to track.

## Native File Formats

Take advantage of existing file formats such as arrow, parquet, duckdb, etc. Unlike git or other VCS that try to be smart with compression, we can leverage the existing file formats that are already highly optimized for the specific use case.

For example, apache arrow is a memory mapped file that makes random access to rows very fast. If we were to compress this data and reconstruct it we would lose the benefits of the memory mapped file.

This is a design tradeoff that is made throughout oxen which makes it less efficient in terms of storage on disk, but easier to integrate with.

Visibility into data is a key design goal of Oxen. Visibility means speed for data to be visible as well, and the less assumptions we make here, the more we can leverage and extend existing file formats.

## Concurrency

* Fearless concurrency
* Hashing data
* Moooooving data over the network
* Moooooving data on disk

## Networking

* Smart Chunking

## Remote Workspaces

Don't download the entire dataset just to contribute.

* oxen workspace add
* oxen workspace commit
* oxen workspace df
* oxen workspace ls

## Compression (Coming Soon)

* [Block level dedup](domains/file_chunk_deduplication.md)
* zlib