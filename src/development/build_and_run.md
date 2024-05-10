# 🔨 Build & Run

## Install Dependencies

Oxen is purely written in Rust 🦀. You should install the Rust toolchain with rustup: https://www.rust-lang.org/tools/install.

```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

If you are a developer and want to learn more about adding code or the overall architecture [start here](docs/dev/AddLibraryCode.md). Otherwise a quick start to make sure everything is working follows.

## Build

Simply use `cargo` within the root directory to build. Cargo looks at `Cargo.toml` to figure out the dependencies and which binaries to build.

```
cargo build
```

Given all the dependencies, the first build can take quite awhile, feel free to go grab some coffee ☕️. Don't worry subsequent builds will be faster and we can speed up the linker later.

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

The library is also used for the Python client (TODO: Link) which should also remain a thin wrapper.

## Run

While developing, the debug binary will be located at `./target/debug/oxen`. It is nice to make a symbolic link to your development build to your `/usr/local/bin` so that you can call the `oxen` binary from anywhere.

```bash
ln -s target/debug/oxen /usr/local/bin/oxen
```

Note: If you have previously installed Oxen through homebrew or other methods make sure it is removed from your $PATH. To verify which `oxen` you are running simply test

```bash
# figure out where oxen is installed
which oxen
# to see if it is a symbolic link
ls -al /path/to/oxen
```

### Speed up the build process

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

# Unit & Integration Tests

Make sure your server is running on the default port and host, then run

*Note:* tests open up a lot of file handles, so limit num test threads if running everything.

You an also increase the number of open files your system allows ulimit before running tests:

```
ulimit -n 10240
```

```
cargo test -- --test-threads=$(nproc)
```

It can be faster (in terms of compilation and runtime) to run a specific test. To run a specific library test:

```
cargo test --lib test_get_metadata_text_readme
```

To run a specific integration test

```
cargo test --test test_rm test_rm_directory_restore_directory
```

To run with all debug output and run a specific test

```
env RUST_LOG=warn,liboxen=debug,integration_test=debug cargo test -- --nocapture test_command_push_clone_pull_push
```

To set a different test host you can set the `OXEN_TEST_HOST` environment variable

```
env OXEN_TEST_HOST=0.0.0.0:4000 cargo test
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
