# binwalk-ng

Firmware analysis tool, built for speed and accuracy.

This crate is a fork of [ReFirmLabs/binwalk](https://github.com/ReFirmLabs/binwalk).

## System Requirements

Building requires the following system packages:

```bash
build-essential liblzma-dev
```

Full extraction support requires additional system and Python dependencies, because
binwalk delegates extraction of many formats to external tools. Run
`dependencies/ubuntu.sh` (Debian/Ubuntu) to install them; see
[dependencies/README.md](https://github.com/binwalk-ng/binwalk-ng/blob/main/dependencies/README.md)
for the other supported platforms.

## Example

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

`Binwalk::builder()` configures the scan:

```rust
use binwalk_ng::Binwalk;

// Only scan for selected signatures
let binwalker = Binwalk::builder()
    .includes(vec!["gzip".to_string(), "lzma".to_string()])
    .build()
    .expect("Failed to build Binwalk");
```

### Cargo Features

Entropy graph generation is enabled by default. To build without it:

```toml
[dependencies]
binwalk-ng = { version = "4", default-features = false }
```

## Links

- [Repository](https://github.com/binwalk-ng/binwalk-ng)
- [Documentation](https://docs.rs/binwalk-ng)
- [Issues](https://github.com/binwalk-ng/binwalk-ng/issues)
