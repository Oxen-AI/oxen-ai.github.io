# Domains

This is a reference for the core domain objects within Oxen. These domains are defined so we are all speaking the same language while diving into the code base. Starting from how objects are stored on disk, to what properties they contain, we will build up intuition of how the system works as a whole.

# File Structure

Similar to `git`, we store all the meta data for a repository in a hidden local `.oxen` directory. To start the learning journey let's initialize and empty Oxen repository locally by using [oxen init](./commands/init.md).

```bash
mkdir my-repo
cd my-repo
oxen init
```

The best way to start learning the architecture and different domain objects is by poking around in this directory.

```bash
ls .oxen
```

You will see a variety of files and folders, including:

```
config.toml
commits/
history/
objects/
refs/
HEAD
```

Let's use these files and folders as a jumping off point to learn about the different domain objects. Since at it's core, Oxen is a version control system, let's start with what is a "version" and how is it represented?

# Config.toml

The first file you will notice is a simple configuration file `config.toml`

TODO: Cleanup the path from here

# Next Up

What is this `commits/` directory and how do we keep track of versions? Let's deep dive into [commit objects](domains/commits.md) next.

