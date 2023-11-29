# Accelerate the Gear node building

## Using `sccache`

Using `sccache` configured with AWS S3 allows decreasing build time.

### Install `sccache`

macOS:

```bash
brew install sccache
```

### Add S3 config

We consider we have already configured AWS S3 storage with the following settings:

- Region: `us-west-1`
- Bucket: `gear-sccache`
- Access key ID: *<AWS_KEY_ID>*
- Secret access key: *<AWS_SECRET_KEY>*

We will use environment variables added to the profile file. For example, if using `zsh`, we need to add the following to the `~/.zshrc`:

```bash
export SCCACHE_BUCKET=gear-sccache
export SCCACHE_REGION=us-west-1
export SCCACHE_S3_KEY_PREFIX=gear
export AWS_ACCESS_KEY_ID=<AWS_KEY_ID>
export AWS_SECRET_ACCESS_KEY="<AWS_SECRET_KEY>"
```

Run to apply these settings immediately:

```bash
source ~/.zshrc
```

### Build Gear node using `sccache`

We need to set the `RUSTC_WRAPPER` environment variable to `sccache`. Also, we need to set `CARGO_INCREMENTAL` to `0` as `sccache` doesn't work well when using incremental building. We can set these environment variables by prepending the `cargo` command in the shell:

```bash
cd gear

# Debug build:
RUSTC_WRAPPER=sccache CARGO_INCREMENTAL=0 cargo b

# Release build:
RUSTC_WRAPPER=sccache CARGO_INCREMENTAL=0 cargo b -r
```

## Using `mold/sold`

`mold` is the linker that uses multiple CPU cores when linking.

### Install `mold/sold`

- macOS

    As `mold` doesn't support macOS we need to build `sold` manually.

    ```bash
    git clone https://github.com/bluewhalesystems/sold.git
    mkdir sold/build
    cd sold/build
    cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_CXX_COMPILER=c++ ..
    cmake --build . -j $(sysctl -n hw.ncpu)
    sudo cmake --build . --target install
    ```

### Configure `cargo` to use `mold/sold`

To use `mold/sold` with Rust, create (or update) `.cargo/config.toml` at your home directory with the following:

```toml
# macOS on M-series:
[target.aarch64-apple-darwin]
rustflags = ["-C", "link-arg=--ld-path=/usr/local/bin/ld64.sold"]
```

Then just run `cargo b [-r]` as usually.

## Using prebuilt `librocksdb.a`

You can use prebuilt static library for the `librocksdb-sys` crate to avoid rebuilding it everytime from C++ sources.

The example below is for macOS:

- Build the `librocksdb-sys` crate in the release profile:

    ```bash
    cargo b -rp librocksdb-sys
    ```
- Find `librocksdb.a` in the target directory:

    ```bash
    find target -name librocksdb.a
    # -> target/release/build/librocksdb-sys-a0a228694895e8fd/out/librocksdb.a
    ```

- Copy `librocksdb.a` to some permanent directory:

    ```bash
    mkdir -p ~/lib
    cp -fv path/to/librocksdb.a ~/lib/
    ```

- Install `bzip2`:

    ```bash
    brew install bzip2
    ```

- Set environment variables:

    ```bash
    export ROCKSDB_STATIC=1
    export ROCKSDB_LIB_DIR=~/lib
    export CARGO_TARGET_AARCH64_APPLE_DARWIN_RUSTFLAGS="-Clink-arg=-L/opt/homebrew/opt/bzip2/lib -Clink-arg=-lbz2"
    ```

- Now the `librocksdb-sys` crate building will be mush faster.

## Avoid rebuilding examples before pallet tests

- Set the `__GEAR_WASM_BUILDER_NO_FEATURES_TRACKING` environment variable to `1`:

    ```bash
    export __GEAR_WASM_BUILDER_NO_FEATURES_TRACKING=1
    cargo t -p pallet-gear
    ```
