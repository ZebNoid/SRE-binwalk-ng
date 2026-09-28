# binwalk-ng

A firmware analysis tool, focused on speed and efficiency. Identifies and extracts files and data embedded inside other files — compressed archives, file systems, boot images, executables, and more.

This repository is a fork of the [ReFirmLabs/binwalk](https://github.com/ReFirmLabs/binwalk) project.

## Features

- Parallel, multi-threaded scanning and (optionally) extracting
- 90+ file signature definitions (firmware, filesystems, archives, media, executables, boot images)
- [Entropy analysis](#entropy-analysis) to detect compression and encryption
- Recursive extraction (matryoshka mode)
- Usable as a Rust library
- JSON output for automation

## Installation

The Docker image is the recommended install; it bundles every system and runtime
dependency, including the external extraction tools.

```bash
docker pull ghcr.io/binwalk-ng/binwalk-ng:main
```

```bash
docker run --rm -v "$PWD:/analysis" ghcr.io/binwalk-ng/binwalk-ng:main -Me firmware.bin
```

To install natively instead, note that binwalk delegates extraction of many formats
to external tools, so those need to be present for extraction to work. The repository
ships scripts that install them:

```bash
git clone https://github.com/binwalk-ng/binwalk-ng
cd binwalk-ng
sudo ./dependencies/ubuntu.sh   # Debian/Ubuntu; see dependencies/README.md for others
cargo install --path .
```

`cargo install --path .` installs the `binwalk` binary to `~/.cargo/bin`, so
make sure that directory is on your PATH. To install the latest release
without cloning, run `cargo install binwalk-ng` instead. If you'd rather not
install it, run it straight from the build directory:
`cargo run --release -- firmware.bin`.

## Usage

```bash
# Scan a file for embedded data
binwalk firmware.bin

# Extract (-e) all signatures found, recursively (-M)
binwalk -Me firmware.bin
```

## Library Usage

Binwalk can be used as a Rust library in your own projects:

```rust
use binwalk_ng::Binwalk;

// Create a new Binwalk instance
let binwalker = Binwalk::new();

// Read in the data to analyze
let file_data = std::fs::read("/tmp/firmware.bin").expect("Failed to read from file");

// Scan the file data and print the results
for result in binwalker.scan(&file_data) {
    println!("{:#?}", result);
}
```

Use the builder to narrow the scan, add your own signatures, or control how files
are read:

```rust
use binwalk_ng::Binwalk;

// Only scan for the signatures you care about
let binwalker = Binwalk::builder()
    .includes(vec!["gzip".to_string(), "lzma".to_string()])
    .build()
    .expect("Failed to build Binwalk");

// Skip signatures you don't want
let binwalker = Binwalk::builder()
    .exclude("jpeg")
    .exclude("png")
    .build()
    .expect("Failed to build Binwalk");

// Search for short signatures at every offset, not just offset 0
let binwalker = Binwalk::builder()
    .full_search(true)
    .build()
    .expect("Failed to build Binwalk");
```

Add binwalk-ng to your project:

```bash
cargo add binwalk-ng
```

## Entropy Analysis

Generate an entropy graph to identify regions of unknown compression or encryption.
Entropy plotting is enabled by default, so no extra build flags are needed:

```bash
binwalk -E firmware.bin
```

Or, to save the graph as a PNG file:

```bash
binwalk -E --png entropy.png firmware.bin
```

## Development

### Prerequisites

- Rust toolchain
- Docker (for full test suite)

### Code Quality

This project uses [`prek`](https://prek.j178.dev/) for Git pre-commit hooks. To start using the hooks, after installing `prek` run

```bash
prek install
```

### Testing

Tests run inside Docker to ensure all external tool dependencies are available.
The test suite depends on the `tests/testdata` submodule, so fetch it first:

```bash
git submodule update --init --recursive
```

Then build the dev image and run the tests:

```bash
docker build --target dev --tag binwalk-ng:dev .
docker run --rm -v "$(pwd):/tmp/binwalk" -e INSTA_UPDATE=new binwalk-ng:dev cargo insta test --unreferenced=reject
```
