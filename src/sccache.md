# Using `sccache` with AWS S3

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

## Measurements

We will measure the build time on the [v1.0.1](https://github.com/gear-tech/gear/releases/tag/v1.0.1) tag of the `gear` repository:

Preparation before the build time measurements:

```bash
rm -rf target
git checkout tags/v1.0.1 -b release-v1.0.1
cargo fetch
```

### macOS / M2 / 8 cores

- Debug build without `sccache`: ?m ??s
- Release build without `sccache`: ?m ??s
- Debug build with `sccache`: **?m ??s**
- Release build with `sccache`: **?m ??s**

### macOS / M3 Pro / 12 cores

- Debug build without `sccache`: 2m 35s
- Release build without `sccache`: 3m 39s
- Debug build with `sccache`: **4m 36s**
- Release build with `sccache`: **3m 47s**
