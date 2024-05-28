# 🐂 🌾 Oxen.ai

Welcome to the Herd! This is a whirlwind tour of the Oxen.ai codebase. This is an evolving artifact meant to document the tool and codebase.

Each section dives into a different part of the code base as well as the file formats on disk. It is a resource for engineers who want to contribute or extend the tooling, or simply want to learn the inner workings.

# What is Oxen?

Oxen at it's core is a blazing fast data version control tool written in Rust. It is optimized for large machine learning datasets. These datasets could consist of many small files (think an images/ folder for computer vision tasks), a few large files (a collection of timeseries datasets as csvs), or many large files (an LLM pre-training dataset of parquet files).

In the [Git Book](https://git-scm.com/book/en/v2/Getting-Started-About-Version-Control) they define "Version Control" as "a system that records changes to a file or set of files over time so that you can recall specific versions later." As a software engineer we typically use tools such as `git` to version our source code. This allows us to keep every single version of a file so that we can revert back to a previous state and compare changes over time. While `git` is great for versioning smaller assets such as files in a code base, it struggles to version large datasets.

# Why build Oxen?

As machine learning engineers, we were frustrated with the speed of managing and iterating on datasets that traditionally would not fit well into `git`. There are extensions to `git` such as `git-lfs` but they are like fitting a square peg in a round hole and come with their own issues. 

Data versions should be easy to interact with locally, fast to sync to a remote, seamless contribute to, and feel like you have terrabytes accessible at your fingertips by slicing and downloading subsets locally when you need it.

# Why write this book?

"What I cannot create, I do not understand". - Richard Feynman

When it comes to open source contribution and scaling up a software project this is true as well. This book is for developers to get an understanding of the internals, design decisions, and places for improvement in the Oxen.ai code base. Open source is meant to not only be open, but understandable. This is an evolving artifact meant to document the tool and codebase.

The concepts listed in this book are not perfect, but are meant to be a guide posts for the current implementation. Along the way we will point out areas for improvement. If you get to a section and think "Why do we do this? The HAS to be a better way." you are probably right! Check out [improvements](./improvements.md) for some ideas we already have, and feel free to add your own.

# Why is Oxen fast?

This is always one of the first questions we get. The simple answer is that there are many [optimizations](./optimizations.md) that make Oxen fast. Many are just fundamental computer science concepts but when stacked together make a nice developer experience for iterating on datasets.

# Why the name Oxen?

"Oxen" 🐂 comes from the fact that the tooling will plow, maintain, and version your data like a good farmer tends to their fields 🌾. During the agricultural revolution the Ox allowed humans to automate the process of plowing fields so they could specialize in higher level tasks. Data is the lifeblood of ML/AI. Let Oxen take care of the grunt work of your infrastructure so you can focus on the higher-level problems that matter to your product.

# Where to start?

First you will want to [install Oxen](./development/installation.md). Once you have the tool up and running, we can dive into the implementation details.

In order to fully grok the Oxen codebase, it's important to define a few terms and understand the different [domain objects](domains.md). This way you'll have the right terminology to build upon and know where to look when adding or debugging features. Let's start by learning about [repositories](./domains/repository.md).
