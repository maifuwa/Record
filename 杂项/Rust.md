# install
rustup `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`
gcc `sudo apt install build-essential`
## build to windows
install windows cc link `sudo apt install mingw-w64`
add cc link to rustup `rustup target add x86_64-pc-windows-gnu`
use cargo to build `cargo build --target x86_64-pc-windows-gnu`

> 环境为 wsl Linux/Ubuntu 24.04.1 LTS
## version
`rustc --version` see version
`rustup update` update version
`rustup self uninstall` uninstall rust
# Cargo
`cargo new <project_name>` create new projcet

`cargo run` will be executed after `cargo bulid`
> if need to pass args to `cargo run`, like `cargo run -- <first_arg> <sendons-arg> <...>`

`cargo clean` to clean target

`cargo add <package_name>` to add package on `Cargo.toml`
> need to install `cargo-edit` just executed `cargo install cargo-edit`

`cargo doc` to create HTML doc
> `--open` auto open in default browser
> `--no-deps` not add lib to doc
> rust will set `private` to all things in default, if want to see code on doc should add `pub` before the code

