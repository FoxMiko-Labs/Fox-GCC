# Fox-GCC

<div align="center">

![Fox-GCC](https://img.shields.io/badge/Fox--GCC-17.0.0-orange?style=for-the-badge&logo=gnu)
![GCC](https://img.shields.io/badge/GCC-Bleeding%20Edge-blue?style=for-the-badge)
![PGO](https://img.shields.io/badge/PGO-Enabled-brightgreen?style=for-the-badge)
![LTO](https://img.shields.io/badge/LTO-Enabled-brightgreen?style=for-the-badge)
![Platform](https://img.shields.io/badge/Platform-Linux%20x86__64-lightgrey?style=for-the-badge&logo=linux)
![License](https://img.shields.io/badge/License-GPLv3-red?style=for-the-badge)

**A bleeding edge, highly optimized GCC cross-compiler toolchain for building Android kernels.**  
Built from GCC master branch with full PGO + LTO optimization pipeline.

[Download](#download) · [Usage](#usage) · [Build Info](#build-information) · [Changelog](../../releases)

</div>

---

## What is Fox-GCC?

Fox-GCC is a custom-built GCC cross-compiler toolchain compiled directly from the **GCC master branch** (bleeding edge), featuring a full two-phase **Profile Guided Optimization (PGO)** pipeline combined with **Link-Time Optimization (LTO)**. It provides both `aarch64-linux-gnu` (AArch64/64-bit) and `arm-eabi` (ARM32/32-bit) cross compilers for building Android kernels.

Built using [mvaisakh/gcc-build](https://github.com/mvaisakh/gcc-build) as the build system base, with vendor string customized to **FoxeGCC** for easy identification.

> Fox-GCC is best paired with [FoxeClang/FoxeReborn](https://github.com/Michikoextv2/FoxeClang) as `CROSS_COMPILE` and `CROSS_COMPILE_ARM32` provider when using Clang as the primary compiler.

---

## Build Information

| Property | Value |
|---|---|
| GCC version | 17.0.0 20260822 (Bleeding Edge) |
| Source branch | GCC master |
| Build date | 2026-08-22 |
| Host | Linux x86_64 |
| Build system | [mvaisakh/gcc-build](https://github.com/mvaisakh/gcc-build) |
| AArch64 target | `aarch64-linux-gnu` |
| ARM32 target | `arm-eabi` |

---

## Optimizations

Fox-GCC is built with a full optimization pipeline applied to the GCC compiler binary itself:

### Profile Guided Optimization (PGO) — 2 Phase
Fox-GCC uses a proper two-phase PGO build process:

**Phase 1 — Instrument:** GCC is compiled with profiling instrumentation (`-fprofile-generate`). The instrumented compiler runs and generates real execution profile data based on actual compilation workloads.

**Phase 2 — Optimize:** GCC is rebuilt from scratch using the collected profile data (`-fprofile-use -fprofile-correction`). The compiler now has precise knowledge of which code paths are hot, enabling smarter decisions about inlining, branch layout, and register allocation.

> Two-phase PGO yields **10–20% faster compile times** compared to a standard build.

### Link-Time Optimization (LTO)
Applied at maximum compression level (`-flto -flto-compression-level=10`), enabling whole-program analysis and cross-module optimizations across the entire GCC binary during the link stage.

> LTO adds another **3–5% improvement** on top of PGO.

### Additional Flags
```
-O3                  Maximum optimization level
-pipe                Use pipes instead of temp files (faster build)
-ffunction-sections  Place each function in its own section
-fdata-sections      Place each data item in its own section
```

---

## Download

Two separate packages are provided — download both for a complete toolchain:

| File | Target | Use |
|---|---|---|
| `FoxGCC-17-arm64gnu-YYYYMMDD.tar.zst` | `aarch64-linux-gnu` | AArch64 / 64-bit kernel |
| `FoxGCC-17-arm-YYYYMMDD.tar.zst` | `arm-eabi` | ARM32 / 32-bit kernel |

Grab the latest release from the [Releases](../../releases) page.

```bash
# Download both packages
wget https://github.com/Michikoextv2/Fox-GCC/releases/download/vXX.X-YYYYMMDD/FoxGCC-17-arm64gnu-YYYYMMDD.tar.zst
wget https://github.com/Michikoextv2/Fox-GCC/releases/download/vXX.X-YYYYMMDD/FoxGCC-17-arm-YYYYMMDD.tar.zst

# Extract
tar --zstd -xf FoxGCC-17-arm64gnu-YYYYMMDD.tar.zst
tar --zstd -xf FoxGCC-17-arm-YYYYMMDD.tar.zst

# Add to PATH
export PATH="/path/to/gcc-arm64gnu/bin:/path/to/gcc-arm/bin:$PATH"
```

Verify:

```bash
aarch64-linux-gnu-gcc --version
# aarch64-linux-gnu-gcc (FoxeGCC) 17.0.0 20260822 (Bleeding Edge)

arm-eabi-gcc --version
# arm-eabi-gcc (FoxeGCC) 17.0.0 20260822 (Bleeding Edge)
```

---

## Usage

### With Clang as Primary Compiler (Recommended)

The recommended setup pairs [FoxeReborn](https://github.com/Michikoextv2/FoxeClang) (Clang) as the primary compiler with Fox-GCC providing cross-compile support:

```bash
export PATH="/path/to/FoxeReborn/bin:/path/to/gcc-arm64gnu/bin:/path/to/gcc-arm/bin:$PATH"

make -j$(nproc) \
    O=out \
    ARCH=arm64 \
    CC=clang \
    LD=ld.lld \
    AR=llvm-ar \
    NM=llvm-nm \
    OBJCOPY=llvm-objcopy \
    OBJDUMP=llvm-objdump \
    STRIP=llvm-strip \
    CLANG_TRIPLE=aarch64-linux-gnu- \
    CROSS_COMPILE=aarch64-linux-gnu- \
    CROSS_COMPILE_ARM32=arm-eabi- \
    LLVM=1 \
    LLVM_IAS=1
```

### GCC Only

To use Fox-GCC as the sole compiler without Clang:

```bash
export PATH="/path/to/gcc-arm64gnu/bin:/path/to/gcc-arm/bin:$PATH"

make -j$(nproc) \
    O=out \
    ARCH=arm64 \
    CROSS_COMPILE=aarch64-linux-gnu- \
    CROSS_COMPILE_ARM32=arm-eabi- \
    CROSS_COMPILE_COMPAT=arm-eabi-
```

> **Note:** GCC only mode works best with kernel 5.x and above. For older kernels (4.x), using Clang as primary compiler with Fox-GCC as cross-compile provider is recommended.

---

## Toolchain Contents

### gcc-arm64gnu (AArch64)
```
gcc-arm64gnu/
└── bin/
    ├── aarch64-linux-gnu-gcc        # GCC AArch64 compiler
    ├── aarch64-linux-gnu-g++        # G++ AArch64 compiler
    ├── aarch64-linux-gnu-ld         # Linker
    ├── aarch64-linux-gnu-ar         # Archiver
    ├── aarch64-linux-gnu-objcopy    # Object copy
    ├── aarch64-linux-gnu-objdump    # Object dump
    └── aarch64-linux-gnu-strip      # Strip tool
```

### gcc-arm (ARM32)
```
gcc-arm/
└── bin/
    ├── arm-eabi-gcc                 # GCC ARM32 compiler
    ├── arm-eabi-g++                 # G++ ARM32 compiler
    ├── arm-eabi-ld                  # Linker
    ├── arm-eabi-ar                  # Archiver
    ├── arm-eabi-objcopy             # Object copy
    ├── arm-eabi-objdump             # Object dump
    └── arm-eabi-strip               # Strip tool
```

---

## Compatibility

| Kernel base | GCC Only | Clang + Fox-GCC |
|---|---|---|
| Linux Mainline (6.x) | ✅ | ✅ |
| Android Common Kernel (ACK) | ✅ | ✅ |
| Qualcomm CAF / CLO (5.x) | ✅ | ✅ |
| Qualcomm CAF / CLO (4.x) | ⚠️ Patch needed | ✅ |
| Samsung Exynos | ✅ | ✅ |
| MediaTek | ✅ | ✅ |

---

## Building from Source

```bash
# Clone build system
git clone https://github.com/mvaisakh/gcc-build.git
cd gcc-build

# Rename vendor string
sed -i 's/--with-pkgversion="Eva GCC"/--with-pkgversion="FoxeGCC"/' build-gcc.sh
sed -i 's/--with-pkgversion="Eva Binutils"/--with-pkgversion="FoxeGCC Binutils"/' build-gcc.sh

# Phase 1 — Instrument (generate PGO profile)
./build-gcc.sh -a arm64gnu -p instrument
./build-gcc.sh -a arm -p instrument

# Phase 2 — Optimize (build with PGO profile)
./build-gcc.sh -a arm64gnu -p optimize
./build-gcc.sh -a arm -p optimize

# Verify
./gcc-arm64gnu/bin/aarch64-linux-gnu-gcc --version
./gcc-arm/bin/arm-eabi-gcc --version
```

> **Requirements:** Linux x86_64, RAM >8GB, ~20GB free disk space, flex, bison, texinfo, gperf, libtool, automake, libgmp-dev, libmpc-dev, libmpfr-dev.

---

## Related Projects

| Project | Description |
|---|---|
| [FoxeClang / FoxeReborn](https://github.com/Michikoextv2/FoxeClang) | Clang/LLVM toolchain — recommended primary compiler |
| [FoxeGCC](https://github.com/Michikoextv2/FoxeGCC) | Alternative GCC build from release tarballs |

---

## Credits

- [Vaisakh (mvaisakh)](https://github.com/mvaisakh/) — gcc-build script
- [GCC Project](https://gcc.gnu.org/) — compiler source
- [GNU Binutils](https://www.gnu.org/software/binutils/) — binary utilities

---

## License

Fox-GCC is built from [GCC](https://gcc.gnu.org/) which is licensed under the [GNU General Public License v3](https://www.gnu.org/licenses/gpl-3.0.html).  
Binutils is also licensed under [GPLv3](https://www.gnu.org/licenses/gpl-3.0.html).
