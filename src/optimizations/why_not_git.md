# Why not write an extension for Git?

We often get asked "Why not use Git-LFS?" and why write an entire Version Control System (VCS) from scratch?

The short answer is developer experience. Owning the tool means owning the end to end developer experience. The plugins and extensions to git that we have tried in the path feel like they were shoe horned in and clunky.

There are certain properties of git that make it great for versioning code, but fall short when it comes to data. Git is really good at huge histories of small text files, like code bases. Where git does not shine is binary files. Many datasets consist of binary files such as images, video, audio, parquet files, etc which are not efficiently stored in git.

There are extensions to git such as git-LFS that work okay when you have one or two large assets that are complimentary to your code, but they are still extremely slow when it comes to versioning large sets of large binary files.

This section dives into some of the reasons why we did not want to use Git-LFS and wrote an entire VCS from scratch.

# What git is missing (could be a good blog post title)

lorem ipsum...

# LFS is More Complexity

The first reason is purely the mental model users have to keep in their head. 

To quote [Gregory Szorc](https://gregoryszorc.com/blog/2021/05/12/why-you-shouldn%27t-use-git-lfs/): 

*"LFS is more complex for Git end users.*

*Git users have to install, configure, and sometimes know about the existence of Git LFS. Version control should just work. Large file handling should just work. End-users shouldn't have to care that large files are handled slightly differently from small files."*

# Push / Pull Bottleneck

How do I atomically add one file without pulling everything? Even further...how do I modify one row or column?

Ex) you update test.csv, I update README.md, we can both push without worrying about pulling first.

Ex) Updating data frame directly with duckdb

Do we call this a VCS-DB or some new term? Allows for higher throughput writes while allowing you to snapshot and diff and query the history.

# Network Protocols

Git sequentially reads and writes objects via [packfiles](https://dev.to/calebsander/git-internals-part-3-the-ssh-transport-2m5c) over the network to create your local and remote copies of the repository. This is inefficient for large files, resulting in slow uploads / downloads of data that is not text.

# SHA-256 is Slow

In order to tell if a file is changed, version control systems use a hashing function of the data. By default git uses SHA-256. This is a relatively slow hashing algorithm. By contrast Oxen uses a faster hashing algorithm called [xxHash](https://github.com/Cyan4973/xxHash). This results in hashing speeds of 31 GB/s for xxHash vs 0.8 GB/s of SHA-256.

The small change in hashing speeds results in an improvement in terms of developer experience when it comes to larger datasets.

If you want to see this in action, simply run `git add` on a large directory of data vs `oxen add` and see the difference in time.

# Git Status Spews All Files

Purely from Developer Experience this is not great. What if you add 100k images in a single commit? It's not practical to have git status show you all 100k files that are added.

With datasets we are dealing more with distributions of data, not individual data points.

# Downloading Full History

Git by default downloads the entire history of the repository on clone. When it comes to datasets, I may only want to download the latest version of a file, not the entire history.

Oxen gives you the flexibility to download just what you need at the time of training, inference, or testing.

# Removing Objects

Removing objects from git is a bit complex because of the references that are made through out the packfiles. Indices have to be recomputed and the history of the repository has to be rewritten if you want to remove a single file.

https://git-scm.com/book/en/v2/Git-Internals-Maintenance-and-Data-Recovery

> There are a lot of great things about Git, but one feature that can cause issues is the fact that a git clone downloads the entire history of the project, including every version of every file. This is fine if the whole thing is source code, because Git is highly optimized to compress that data efficiently. However, if someone at any point in the history of your project added a single huge file, every clone for all time will be forced to download that large file, even if it was removed from the project in the very next commit. Because it’s reachable from the history, it will always be there.

# Packfiles

One of the ways that git can save space is by using delta encoding to store only the differences between files. They do this through the use of [packfiles](https://dev.to/calebsander/git-internals-part-2-packfiles-1jg8). The objects are not stored directly in the objects directory rather packed up together in a packfile to make data transfer and compression easier.

Within a pack file there are multiple ways an object can be stored.

This is great for codebases, but not optimal for binary files.

# Sources

### Git SCM book

* https://git-scm.com/book/en/v2/Git-Internals-Maintenance-and-Data-Recovery

### Avoid Git-LFS if Possible

* https://news.ycombinator.com/item?id=27134972
* https://gregoryszorc.com/blog/2021/05/12/why-you-shouldn%27t-use-git-lfs/

### Dev.to git-internals

* https://dev.to/calebsander/git-internals-part-2-packfiles-1jg8

### Pack files

Git’s database internals I: packed object store

* https://github.blog/2022-08-29-gits-database-internals-i-packed-object-store/#delta-compression