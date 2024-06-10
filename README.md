# electron for riscv64

This repo is a fork of electron with riscv64 support added.

Status: Work In Progress

**Note**: [A performance regression](https://github.com/riscv-forks/electron/issues/1) is fixed recently and only the latest versions and v29.4.0 are rebuilt. Please upgrade if you still using other versions.

Releases are published in https://github.com/riscv-forks/electron-riscv-releases

## Repo Organization

Branches with riscv64 support have a suffix of `-riscv`. e.g. `main-riscv`, `28-x-y-riscv`, `v29.1.1-riscv`.

## Necessary Toolchains for cross-compilation

- Debian riscv sysroot
- Clang toolchain
- Rust toolchain

Prebuilt toolchains are available in GitHub Releases. They can also be built from source by corresponding scripts, e.g.
`//tools/rust/build_rust.py`.

**Note**: To build the sysroot, please checkout chromium 121.0.6167.90 source code and apply the patch. The sysroot creator patch is not maintained in electron patches.

Toolchain issues:

- https://github.com/llvm/llvm-project/issues/79944 (This only affects building chromiums but coincidentally doesn't affect building electron)
- [Rust doesn't embed target-abi llvm module flag](https://github.com/rust-lang/rust/issues/121924). There's a work-around applied to rust toolchain. It has been fixed in rust by https://github.com/rust-lang/rust/pull/123612

## Build Instruction

This repo supports both building in RISC-V environment and cross-compiling from a x86-64 host.

### Cross-Compile

#### Host Environment

Debian bookworm.

#### Getting the Source

First, [install depot_tools and add it to PATH](https://commondatastorage.googleapis.com/chrome-infra-docs/flat/depot_tools/docs/html/depot_tools_tutorial.html#_setting_up).

Then, get the source code:

```bash
$ mkdir electron && cd electron
$ gclient config --name "src/electron" --unmanaged https://github.com/riscv-forks/electron@BRANCH_NAME_HERE
$ gclient sync --with_branch_heads --with_tags
```

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

Arch Linux riscv64 is building electron on riscv64. 

The patch of Arch Linux's PKGBUILD is available here for reference: https://github.com/felixonmars/archriscv-packages/blob/master/electron28/riscv64.patch

## TODOs

- [ ] Upstream more patches.
- [ ] Set up CI to automatically create riscv branches from upstream tags.
