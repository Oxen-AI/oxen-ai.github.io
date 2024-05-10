# Optimizations

There are many optimizations that make Oxen go brrr. From the core merkle tree structure, to hashing protocol, to networking, to interacting with remote datasets. Oxen is meant to make it feel like you have terrabytes of data at your fingertips whatever machine you are on.

# ~TLDRs~

We often get asked: "What makes Oxen different from other VCS?"

Without diving into the gnitty gritty details, here are some highlights. If you want to go deeper, don't worry, we also dive deep into the implementation details of each throughout the book.

## Merkle Tree

* Downloading Sub Trees
* Per Folder Sub Trees
* Block Level Dedup
* Only download latest
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

## Compression

* Block level dedup
* zlib

## Concurrency

* Fearless concurrency
* Hashing data
* Moooooving data over the network
* Moooooving data on disk

## Networking

* Smart Chunking

## Remote Workspaces

* oxen remote add
* oxen remote commit
* oxen remote df
* oxen remote ls