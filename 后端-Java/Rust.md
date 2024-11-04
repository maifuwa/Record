
# install
rustup `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`
gcc `sudo apt install build-essential`
## build to windows
install windows cc link `sudo apt install mingw-w64`
add cc link to rustup `rustup target add x86_64-pc-windows-gnu`
use cargo to build `cargo build --target x86_64-pc-windows-gnu`

> 环境为 wsl Linux/Ubuntu 24.04.1 LTS

# Cargo
create new projcet `cargo new <project_name>`

`cargo run` will be executed after `cargo bulid`

if need to pass args to `cargo run`, like `cargo run -- <first_arg> <sendons-arg> <...>`

