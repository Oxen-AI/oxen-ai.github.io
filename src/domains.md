# Domain Objects

In order to fully grok the Oxen codebase, it's important to define a few terms and understand the different [domain objects](domains.md). This way you'll have the right terminology to build upon and know where to look when adding or debugging features. 

These domains are defined so we are all speaking the same language while diving into the code base. We will start with what the objects are, why they exist, and how objects are stored on disk, then we will build up intuition of how the system works as a whole.

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
commits/
history/
objects/
refs/
HEAD
config.toml
```

Let's use these files and folders as a jumping off point to learn about the different domain objects. 

# First Up: Repositories

All of the domain objects exist within the context of a "Repository", so let's [start there](domains/repositories.md). All of the files and folders within the `.oxen` directory represent different sub components of a Repository, but we need some over arching objects to kick the process all off. These are what we call the `LocalRepository` and `RemoteRepository`.


