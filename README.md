# electron for riscv64

This repo is a fork of electron with riscv64 support added.

Releases are published in https://github.com/riscv-forks/electron-riscv-releases

## Repo Organization

Branches with riscv64 support have a suffix of `-riscv`. e.g. `main-riscv`, `28-x-y-riscv`, `v29.1.1-riscv`.

## Necessary Toolchains for cross-compilation

- Debian riscv sysroot

**Note**: To build the sysroot, please checkout chromium source code at `1273786fb3efd` and apply the `sysroot.patch`. The sysroot creator patch is not maintained in electron patches.

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

#### Sysroot Preparation

Please download the latest debian sysroot from GitHub Releases and put it at `build/linux/debian_trixie_riscv64-sysroot`:

```bash
rm -rf build/linux/debian_trixie_riscv64-sysroot
mkdir build/linux/debian_trixie_riscv64-sysroot
tar xf "$YOUR_DOWNLOADED_SYSROOT_FILE" --directory=build/linux/debian_trixie_riscv64-sysroot
```

#### Building from Source

Run GN:

```bash
./buildtools/linux64/gn gen out/Release-riscv64 --args="import(\"//electron/build/args/release.gn\") target_cpu = \"riscv64\""
```

Run ninja:

```bash
autoninja -C out/Release-riscv64/ electron
```

Generate Dist Zip:

```bash
electron/script/strip-binaries.py -d out/Release-riscv64 --target-cpu riscv64
autoninja -C out/Release-riscv64 electron:electron_dist_zip
```

### Build on RISC-V

Arch Linux riscv64 is building electron on riscv64. 

The patch of Arch Linux's PKGBUILD is available here for reference: https://github.com/felixonmars/archriscv-packages/blob/master/electron28/riscv64.patch

## TODOs

- [ ] Set up CI to automatically create riscv branches from upstream tags.
- [ ] Cover more tests in Chromium CI.
