# Gear for Rustaceans

[![Publish Status](https://github.com/gear-tech/gear-tech.github.io/workflows/Publish/badge.svg)](https://github.com/gear-tech/gear.rs/actions/workflows/publish.yml?query=branch%3Amaster)

Gear projects navigator for Rust developers.

⚙️🦀 https://gear.rs

## Build Locally

0. Install Rust language using [rustup](https://rustup.rs/).

1. Clone:

    ```bash
    git clone https://github.com/gear-tech/gear-tech.github.io.git
    cd gear-tech.github.io
    ```

2. Install [mdBook](https://github.com/rust-lang/mdBook):

    ```bash
    cargo install mdbook
    ```

3. Edit source files then save.

4. Build:

    ```bash
    mdbook build
    ```

5. Serve:

    ```bash
    mdbook serve
    ```

6. Open in browser: http://localhost:3000/

## License

The content on gear.rs is distributed under the [CC0 1.0 Universal License](LICENSE).
