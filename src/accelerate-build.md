# Accelerate the Gear node building

## Using `sccache`

Using `sccache` configured with AWS S3 allows decreasing build time.

## Install `sccache`

macOS:

```bash
brew install sccache
```

## Add S3 config

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

## Build Gear node using `sccache`

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

## Install `mold/sold`

### macOS

As `mold` doesn't support macOS we need to build `sold` manually.

```bash
git clone git@github.com:bluewhalesystems/sold.git
mkdir sold/build
cd ./sold/build
# if you don't have cmake, please run:
# brew install cmake
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_CXX_COMPILER=c++ ..
cmake --build . -j $(sysctl -n hw.ncpu)
sudo cmake --build . --target install
```

## Configure `cargo` to use `mold/sold`

To use `mold/sold` with Rust, create (or update) `.cargo/config.toml` at your home directory with the following:

```toml
# macOS on M-series:
[target.aarch64-apple-darwin]
rustflags = ["-C", "link-arg=--ld-path=/usr/local/bin/ld64.sold"]
```

Then just run `cargo b [-r]` as usually.

## Measurements

We will measure the build time on the [v1.0.1](https://github.com/gear-tech/gear/releases/tag/v1.0.1) tag of the `gear` repository:

Preparation before the build time measurements:

```bash
rm -rf target
git checkout tags/v1.0.1 -b release-v1.0.1
cargo fetch
```

### macOS / M2 / 8 cores

- Debug profile by defaul: 4m 09s
- Release profile by defaul: 6m 12s
- Debug profile with `sccache`: **4m 47s**
- Release profile with `sccache`: **4m 26s**

### macOS / M3 Pro / 12 cores

- Debug profile by default: 2m 35s
- Release profile by defaul: 3m 39s
- Debug profile with `sccache`: **4m 36s**
- Release profile with `sccache`: **3m 47s**
