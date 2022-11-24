# wasmvm

This is a wrapper around the [CosmWasm VM](https://github.com/CosmWasm/cosmwasm/tree/main/packages/vm).
It allows you to compile, initialize and execute CosmWasm smart contracts
from Go applications, in particular from [x/wasm](https://github.com/CosmWasm/wasmd/tree/master/x/wasm).

## Structure

This repo contains both Rust and Go code. The rust code is compiled into a dll/so
to be linked via cgo and wrapped with a pleasant Go API. The full build step
involves compiling rust -> C library, and linking that library to the Go code.
For ergonomics of the user, we will include pre-compiled libraries to easily
link with, and Go developers should just be able to import this directly.

## Supported Platforms

Requires Rust 1.55+ and Go 1.17+.

The Rust implementation of the VM is compiled to a library called libwasmvm. This is
then linked to the Go code when the final binary is built. For that reason not all
systems supported by Go are supported by this project.

Linux (tested on Ubuntu, Debian, and CentOS7, Alpine) and macOS is supported.
We are working on Windows (#288).

[#288]: https://github.com/CosmWasm/wasmvm/pull/288

### Builds of libwasmvm

Our system currently supports the following builds. In general we can only support targets
that are [supported by Wasmer's singlepass backend](https://docs.wasmer.io/ecosystem/wasmer/wasmer-features#compiler-support-by-chipset),
which for example excludes all 32 bit systems.

<!-- AUTO GENERATED BY libwasmvm_builds.py START -->

| OS family       | Arch    | Linking | Supported                    | Note                                                                                                                                   |
| --------------- | ------- | ------- | ---------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| Linux (glibc)   | x86_64  | shared  | ✅​libwasmvm.x86_64.so       |                                                                                                                                        |
| Linux (glibc)   | x86_64  | static  | 🚫​                          | Would link libwasmvm statically but glibc dynamically as static glibc linking is not recommended. Potentially interesting for Osmosis. |
| Linux (glibc)   | aarch64 | shared  | ✅​libwasmvm.aarch64.so      |                                                                                                                                        |
| Linux (glibc)   | aarch64 | static  | 🚫​                          |                                                                                                                                        |
| Linux (musl)    | x86_64  | shared  | 🚫​                          | Possible but not needed                                                                                                                |
| Linux (musl)    | x86_64  | static  | ✅​libwasmvm_muslc.x86_64.a  |                                                                                                                                        |
| Linux (musl)    | aarch64 | shared  | 🚫​                          | Possible but not needed                                                                                                                |
| Linux (musl)    | aarch64 | static  | ✅​libwasmvm_muslc.aarch64.a |                                                                                                                                        |
| macOS           | x86_64  | shared  | ✅​libwasmvm.dylib           | Fat/universal library with multiple archs ([#294])                                                                                     |
| macOS           | x86_64  | static  | 🚫​                          |                                                                                                                                        |
| macOS           | aarch64 | shared  | ✅​libwasmvm.dylib           | Fat/universal library with multiple archs ([#294])                                                                                     |
| macOS           | aarch64 | static  | 🚫​                          |                                                                                                                                        |
| Windows (mingw) | x86_64  | shared  | 🏗​wasmvm.dll                 | See [#288]                                                                                                                             |
| Windows (mingw) | x86_64  | static  | 🚫​                          |                                                                                                                                        |
| Windows (mingw) | aarch64 | shared  | 🚫​                          |                                                                                                                                        |
| Windows (mingw) | aarch64 | static  | 🚫​                          |                                                                                                                                        |

[#288]: https://github.com/CosmWasm/wasmvm/pull/288
[#294]: https://github.com/CosmWasm/wasmvm/pull/294

<!-- AUTO GENERATED BY libwasmvm_builds.py END -->

## Docs

Run `(cd libwasmvm && cargo doc --no-deps --open)`.

## Design

Please read the [Documentation](./spec/Index.md) to understand both the general
[Architecture](./spec/Architecture.md), as well as the more detailed
[Specification](./spec/Specification.md) of the parameters and entry points.

## Development

There are two halfs to this code - go and rust. The first step is to ensure that there is
a proper dll built for your platform. This should be `api/libwasmvm.X`, where X is:

- `so` for Linux systems
- `dylib` for MacOS
- `dll` for Windows - Not currently supported due to upstream dependency

If this is present, then `make test` will run the Go test suite and you can import this code freely.
If it is not present you will have to build it for your system, and ideally add it to this repo
with a PR (on your fork). We will set up a proper CI system for building these binaries,
but we are not there yet.

To build the rust side, try `make build-rust` and wait for it to compile. This depends on
`cargo` being installed with `rustc` version 1.47+. Generally, you can just use `rustup` to
install all this with no problems.

## Toolchain

We fix the Rust version in the CI and build containers, so the following should be in sync:

- `.circleci/config.yml`
- `builders/Dockerfile.*`

For development you should be able to use any reasonably up-to-date Rust stable.

## Build

make release-build-alpine RUSTFLAGS='-C link-arg=-s' => ec18ef336a5d4448e78d95aad1e9243972e37cd8bae9d7b67f036b0bdbd470b6

make release-build-linux RUSTFLAGS='-C link-arg=-s' => sha256sum: de3bcbdb6130ddd0ecbe472422c9055559788755be5ef9bba3088fc0747c0631 

make release-build-windows RUSTFLAGS='-C link-arg=-s' => sha256sum: 40f863332988f64bb55034504d043c2734f54284bdd832b4b65fb2c823d70021 

make release-build-macos RUSTFLAGS='-C link-arg=-s' => sha256sum: b3352c94e31ae0e5f45608ef9e8e63a6e1b5707526eb38473c1aa65404eba15a