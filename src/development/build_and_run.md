# 🔨 Build & Run

## Install Dependencies

Oxen is purely written in Rust 🦀. You should install the Rust toolchain with rustup: https://www.rust-lang.org/tools/install.

```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

If you are a developer and want to learn more about adding code or the overall architecture [start here](docs/dev/AddLibraryCode.md). Otherwise a quick start to make sure everything is working follows.

## Building from Source

To build the command line tool from source, you can follow these steps.

1. Install rustup via the instructions at https://rustup.rs/
2. Clone the repository https://github.com/Oxen-AI/Oxen
    ```bash
    git clone git@github.com:Oxen-AI/Oxen.git
    ```
3. `cd` into the cloned repository
    ```bash
    cd Oxen
    ```
4. Run this command (the release flag is recommended but not necessary):
    ```bash
    cargo build --release
    ```
5. After the build has finished, the `oxen` binary will be in `Oxen/target/release` (or, if you did not use the --release flag, `Oxen/target/debug`).

    Now, to make it usable from a terminal window, you have the option to add it to create a symlink or to add it to your `PATH`.
6. To add oxen to your `PATH`:

    Add this line to your `.bashrc` (or equivalent, e.g. `.zshrc`)
    ```bash
    export PATH="$PATH:/path/to/Oxen/target/release"
    ```
7. Alternatively, to create a symlink, run the following command:
    ```bash
    sudo ln -s /path/to/Oxen/target/release/oxen /usr/local/bin/oxen
    ```
    Note that if you did not use the `--release` flag when building Oxen, you will have to change the path.

## Library, CLI, Server

There are three components that are built during `cargo build` and they are separated into three directories within the `src` folder.

```bash
ls src
```

```
cli/
lib/
server/
```

The library is all the shared code between the CLI and Server. This contains the majority of classes and business logic. The CLI and Server are meant to be thin wrappers over the core oxen library functionality.

The library is also used for the [Python Client](https://github.com/Oxen-AI/oxen-release) which should also remain a thin wrapper.

## Speed up the build process

You can use
the [mold](https://github.com/rui314/mold) linker to speed up builds (The
commercial Mac OS version is [sold](https://github.com/bluewhalesystems/sold)).

Assuming you have purchased a license, you can use the following instructions to
install sold and configure cargo to use it for building Oxen:

```
git clone https://github.com/bluewhalesystems/sold.git

mkdir sold/build
cd sold/build
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_CXX_COMPILER=c++ ..
cmake --build . -j $(nproc)
sudo cmake --install .
```

Then create `.cargo/config.toml` in your Oxen repo root with the following
content:

```
[target.x86_64-unknown-linux-gnu]
rustflags = ["-C", "link-arg=-fuse-ld=/usr/local/bin/ld64.mold"]

[target.x86_64-apple-darwin]
rustflags = ["-C", "link-arg=-fuse-ld=/usr/local/bin/ld64.mold"]

```

**For macOS with Apple Silicon**, you can use the [lld](https://lld.llvm.org/) linker.

```
brew install llvm
```

Then create `.cargo/config.toml` in your Oxen repo root with the following:

```
[target.aarch64-apple-darwin]
rustflags = [ "-C", "link-arg=-fuse-ld=/opt/homebrew/opt/llvm/bin/ld64.lld", ]

```

## Run Oxen-Server

Generate a config file and token to give user access to the server

```
./target/debug/oxen-server add-user --email ox@oxen.ai --name Ox --output user_config.toml
```

Copy the config to the default locations

```
mkdir ~/.oxen
```

```
mv user_config.toml ~/.oxen/user_config.toml
```

```
cp ~/.oxen/user_config.toml data/test/config/user_config.toml
```

Set where you want the data to be synced to. The default sync directory is `./data/` to change it set the SYNC_DIR environment variable to a path.

```
export SYNC_DIR=/path/to/sync/dir
```

Run the server

```
./target/debug/oxen-server start
```

To run the server with live reload, first install cargo-watch

```
cargo install cargo-watch
```

Then run the server like this

```
cargo watch -- cargo run --bin oxen-server start
```

# CLI Commands

Now feel free to try out some CLI commands and see the tool in action!

```
oxen init .
oxen status
oxen add images/
oxen status
oxen commit -m "added images"
oxen create-remote --name ox/wikipedia --host 0.0.0.0:3001 --scheme http
oxen config --set-remote origin http://localhost:3001/ox/wikipedia
oxen push origin main
```
