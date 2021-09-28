# Building Torrust Tracker

## Required tools
- [Git](https://git-scm.com) - Version Control
- [Rust](https://www.rust-lang.org/) - Compiler toolchain & Package Manager (cargo)

### Getting the sources
```bash
git clone https://github.com/torrust/torrust.git
```

If you prefer to just download the code, you can get the [latest codebase here](https://github.com/torrust/torrust/archive/master.zip).

### Prerequisites
While compiling sqlx will check if queries are valid against a database, but it doesnt create it.
To create it manually you will need to install the sqlx-cli by running:
```bash
cargo install sqlx-cli
```

To create it and run the migrations run:
```bash
sqlx db setup
```

### Building
This step will download all required dependencies (from [crates.io](https://crates.io/)) and build them as well. 

Building should always be done with the latest Rust compiler.

```bash
cd torrust
cargo build --release
```

Once cargo is done building, `torrust` will be built at `target/release/torrust`.