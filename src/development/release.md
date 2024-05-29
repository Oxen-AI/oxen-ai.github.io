# 🐂 Releasing Oxen Into The Wild 🌾

Right now this is mainly for [me](https://www.linkedin.com/in/greg-schoeninger/) to document how I release new versions of the open source Oxen.ai binaries.

If anyone wants to help with the release process, please let me know!

## Bump CLI/Server Versions

For the CLI and Oxen-Server binaries, make sure to update in all Cargo.toml files in our [Oxen-AI/Oxen](https://github.com/Oxen-AI/Oxen) repo.

- [Cargo.toml](https://github.com/Oxen-AI/Oxen/blob/main/Cargo.toml)
- [src/server/Cargo.toml](https://github.com/Oxen-AI/Oxen/blob/main/src/server/Cargo.toml)
- [src/cli/Cargo.toml](https://github.com/Oxen-AI/Oxen/blob/main/src/cli/Cargo.toml)

## Create Tag

We use git tags to kick off CI within GitHub actions.

`git tag -a v$VERSION -m "version $VERSION"`

## Push Tag

Builds will show up in this repositories [releases](https://github.com/Oxen-AI/Oxen/releases) with the tag you just specified.

`git push origin v$VERSION`

## Update Homebrew Install

There are separate homebrew repositories for the `oxen` CLI and the `oxen-server` binary.

[Oxen-AI/homebrew-oxen](https://github.com/Oxen-AI/homebrew-oxen)

[Oxen-AI/homebrew-oxen-server](https://github.com/Oxen-AI/homebrew-oxen-server)

You will need to compute shasum(s) of each release and update the `Formula/*.rb` in both repos above.

Use the [compute_hashes.sh](https://github.com/Oxen-AI/homebrew-oxen/blob/main/compute_hashes.sh) script in [homebrew-oxen](https://github.com/Oxen-AI/homebrew-oxen) repo to compute the shasum(s) of each release.

To verify the formula(s) locally:

```
cd /path/to/homebrew-oxen-server
brew install Formula/oxen.rb
oxen --version
```

```
cd /path/to/homebrew-oxen-server
brew install Formula/oxen-server.rb
oxen-server --version
```

## Update Release Notes

TODO: We need to get better at this. 

Suggestions welcome 🙏.