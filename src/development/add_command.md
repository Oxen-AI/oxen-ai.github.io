# Adding a Command

The main entry point to the Command Line Interface (CLI) is through the [main.rs](https://github.com/Oxen-AI/Oxen/blob/main/src/cli/src/main.rs) file. This file is located in the [Oxen/src/cli/src](https://github.com/Oxen-AI/Oxen/tree/main/src/cli/src) directory.

Each command is defined in it's own submodule and implements the `RunCmd` trait.

```rust
#[async_trait]
pub trait RunCmd {
    fn name(&self) -> &str;
    fn args(&self) -> clap::Command;
    async fn run(&self, args: &clap::ArgMatches) -> Result<(), OxenError>;
}
```

These submodules can be found in `cmd` subdirectory. They are named after the command they implement. For example if you are curious how `oxen add` is implemented, you would look at [add.rs](https://github.com/Oxen-AI/Oxen/blob/main/src/cli/src/cmd/add.rs).

# Moo' World

To show this pattern in action, let's add a new command to Oxen. This new command will be a simple "Hello, World!" command. The new command will be named "moo" and will be implemented in the [moo.rs](https://github.com/Oxen-AI/Oxen/blob/main/src/cli/src/cmd/moo.rs) file.

The command simply prints "moo!" when you run `oxen moo`. It also takes a `--loud` flag which makes it print "MOO!" instead of "moo!" if you pass the flag as well as a `-n` flag which adds extra o's to the end of the string.

```bash
$ oxen moo

moo
```

```bash
oxen moo --loud

MOO!
```

```bash
oxen moo -n 10

moooooooooo
```

# Name The Command

The first method to implement in the trait is simply the name of the command. This is used to identify the command in the CLI and in the help menu.

```rust
impl RunCmd for MooCmd {
    fn name(&self) -> &str {
        "moo"
    }
}
```

# Setup Args

The next step is setting up the command line arguments. We use the [clap](https://docs.rs/clap/latest/clap/) crate to handle the command line arguments. The arguments are defined in the `args` method.

```rust
impl RunCmd for MooCmd {
    fn args(&self) -> Command {
        // Setups the CLI args for the command
        Command::new(NAME)
            .about("Hello, world! 🐂")
            .arg(
                Arg::new("number")
                    .long("number")
                    .short('n')
                    .help("How long is the moo?")
                    .default_value("2")
                    .action(clap::ArgAction::Set),
            )
            .arg(
                Arg::new("loud")
                    .long("loud")
                    .short('l')
                    .help("Make the MOO louder.")
                    .action(clap::ArgAction::SetTrue),
            )
    }
}
```

# Parse Args and Run Command

Finally we need to implement the `run` method which is called when the command is run. The `run` method is called with the parsed command line arguments.

```rust
impl RunCmd for MooCmd {
    async fn run(&self, args: &clap::ArgMatches) -> Result<(), OxenError> {
        // Parse Args
        let n = args
            .get_one::<String>("number")
            .expect("Must supply number")
            .parse::<usize>()
            .expect("number must be a valid integer.");

        let loud = args.get_flag("loud");
        if loud {
            // Print the moo loudly with -n number of o's
            println!("M{}!", "O".repeat(n));
        } else {
            // Print the moo with -n number of o's
            println!("m{}", "o".repeat(n));
        }

        Ok(())
    }
}
```

If a command returns an `OxenError` it will be handled and printed in the `main.rs` file and return a non zero exit code.

# Add to CLI

Now that our command is implemented, we need to add it to the CLI. This is done in the [main.rs](https://github.com/Oxen-AI/Oxen/blob/main/src/cli/src/main.rs) file. All you need to do is add a new instance of your command to the `cmds` vector. The rest of the file is just adding the arguments, parsing them, then calling your `run` method.

```rust
let cmds: Vec<Box<dyn cmd::RunCmd>> = vec![
    Box::new(cmd::AddCmd),
    Box::new(cmd::MooCmd), // Your new command
];

// ... run commands
```

This should be all you need to get Oxen to go "MOO!". Let's build and run.

```bash
cargo build
./target/debug/oxen moo --help
```

You will see the help menu for your new command.

```bash
Hello, world! 🐂

Usage: oxen moo [OPTIONS]

Options:
  -n, --number <number>  How long is the moo? [default: 2]
  -l, --loud             Make the MOO louder.
  -h, --help             Print help
```

Then you can simply run your command.

```bash
./target/debug/oxen moo
```

You should see the output "moo"

```
moo
```

You can also make the moo louder with the `--loud` flag and add more o's with the `-n` flag.

```bash
$ ./target/debug/oxen moo --loud
MOO!
```

```bash
$ ./target/debug/oxen moo -n 10
moooooooooo
```

🎉 And there you have it! 

Congrats on adding your first command to Oxen! The moo command is already implemented in the main Oxen codebase as an easter egg and an example you can follow along with.
