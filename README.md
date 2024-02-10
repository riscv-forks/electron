# electron for riscv64

This repo is a fork of electron with riscv64 support added.

Status: Work In Progress

Releases will be published later.

## Repo Organization

Branches with riscv64 support have a suffix of `-riscv`. e.g. `main-riscv`, `28-x-y-riscv`

## Necessary Toolchains for cross-compilation

- Debian riscv sysroot
- Clang toolchain
- Rust toolchain

Prebuilt toolchains are available in GitHub Releases. They can also be built from source by corresponding scripts, e.g.
`//tools/rust/build_rust.py`.

Toolchain issues:

- https://github.com/llvm/llvm-project/issues/79944
- Rust doesn't embed target-abi llvm module flag. There's a work-around applied to rust toolchain. TODO: offer this to upstream.

## Build Instruction

This repo supports both building in RISC-V environment and cross-compiling from a x86-64 host.

### Cross-Compile

#### Host Environment

Debian bookworm.

#### Toolchain Preparation

Download and unpack riscv64 sysroot:

```
mkdir build/linux/debian_sid_riscv64-sysroot
tar xf ../debian_sid_riscv64_sysroot.tar.xz --directory=build/linux/debian_sid_riscv64-sysroot
mkdir -p third_party/llvm-build-tools/
ln -s ../../build/linux/debian_sid_riscv64-sysroot third_party/llvm-build-tools/debian_sid_riscv64_sysroot
```

Download and unpack clang toolchain built with riscv64 support:

```bash
cd third_party/llvm-build/Release+Asserts
tar xf <path to clang-xxx.tar.xz>
```

Download and unpack rust toolchain built with fixes:

```
cd third_party/
tar xf <path to rust-toolchain-xxx.tar.xz>
cp rust-toolchain/{,INSTALLED_}VERSION
```

Run GN:

```
./buildtools/linux64/gn gen out/Release-riscv64 --args="import(\"//electron/build/args/release.gn\") target_cpu = \"riscv64\" is_clang = true"
```

Run ninja:

```
ninja -C out/Release-riscv64/ electron
```

Generate Dist Zip:

```
electron/script/strip-binaries.py -d out/Release-riscv64 --target-cpu riscv64
ninja -C out/Release-riscv64 electron:electron_dist_zip
```

### Build on RISC-V

TODO

## TODOs

- [ ] Upstream more patches.
- [ ] Set up CI to build and publish toolchains to GitHub Releases.
- [ ] Set up CI to build and publish electron to GitHub Releases
- [ ] Set up CI to automatically create riscv branches from upstream tags.
