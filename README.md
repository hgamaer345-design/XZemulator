# XZemulator

XZemulator is a set of tools, written from scratch, for taking apart iOS firmware images (IPSW files). It is also the start of a user-space iOS emulator built on top of those tools.

**Status, version 0.2.0:** this release contains the firmware toolchain. The emulator itself is still in development and is not part of this release. That covers the AArch64 interpreter, the Mach-O loader, iOS system-call translation, and an iPhone-shaped screen.

## Download

Get the files from the [Releases page](https://github.com/hgamaer345-design/XZemulator/releases).

| Platform | File | Notes |
|---|---|---|
| Linux x86_64 | `XZemulator-0.2.0-x86_64.AppImage` | Runs on glibc 2.17 or newer. Tested on Debian 13. |
| Windows 10 / 11, 64-bit | `XZemulator-0.2.0-Setup.exe` | Installs per user, no administrator rights needed. It passed automated structural checks but has not yet been run on a Windows machine. |

Python is bundled in both, so nothing else needs to be installed.

SHA-256 checksums:

```
f07d193a92d1d6883128de527c3592e45615053853340cbc9ed44cc34cd667f7  XZemulator-0.2.0-x86_64.AppImage
aa3c360685d0daa12c9acaa991bb2ae93d32f5b845c8f6c13acba0fb88707828  XZemulator-0.2.0-Setup.exe
```

## Install

**Linux**

```
chmod +x XZemulator-0.2.0-x86_64.AppImage
./XZemulator-0.2.0-x86_64.AppImage --help
```

For a short command, link the AppImage under a name such as `xzemu`. Do not name the link `xz`, because that would hide the system's xz compression tool. Links named `xz-<tool>` run one tool directly, for example `xz-apfs`.

```
ln -s "$PWD/XZemulator-0.2.0-x86_64.AppImage" ~/.local/bin/xzemu
```

If your system has no FUSE 2 library, add `--appimage-extract-and-run` before the other arguments.

**Windows**

Run `XZemulator-0.2.0-Setup.exe`. It installs into `%LOCALAPPDATA%\XZemulator` and adds that folder to your user PATH. It also creates a Start Menu entry named "XZemulator Console". Open a new console and run `xz --help`. To uninstall, use Settings > Apps.

## Usage

XZemulator does not include any firmware. Download an IPSW for your own device and unzip it, since an IPSW is a ZIP archive. Then point the tools at the files inside. These examples use files from an iPhone 15 Pro Max IPSW for iOS 27.0.1. They use the Windows command name `xz`; on Linux, use `xzemu` from the install step or the AppImage's own path.

```
xz img4 info kernelcache.release.iphone16                         # what kind of payload, how it is compressed
xz img4 extract kernelcache.release.iphone16 kernelcache.macho    # decompresses LZFSE to a 73 MB Mach-O
xz macho fileset ls kernelcache.macho                             # the kernel and its 263 extensions
xz arm64 dis kernelcache.macho --seg __TEXT_EXEC --len 64         # disassemble kernel code
xz devicetree dump Firmware/all_flash/DeviceTree.d84ap.im4p       # the hardware description
xz plistdump summary BuildManifest.plist                          # versions, device, every firmware part
xz dmg extract 043-69297-793.dmg ramdisk.raw                      # disk image to raw APFS
xz apfs ls ramdisk.raw /                                          # browse the file system
xz apfs extract ramdisk.raw / ramdisk_files                       # copy every file out, decompressed
xz selftest                                                       # check every tool works
```

Every tool has its own help, for example `xz help apfs`.

| Tool | What it reads |
|---|---|
| `img4` | IMG4, IM4P and IM4M firmware containers, including LZFSE and LZSS payloads |
| `lzfse` | Apple's LZFSE and LZVN compression, using a decoder written in C |
| `dmg` | UDIF disk images (.dmg) and raw images, with partition tables |
| `apfs` | APFS file systems, read only, including compressed files, hard links and symlinks |
| `macho` | Mach-O binaries, fileset kernelcaches, chained fixups, symbols, entitlements |
| `arm64` | AArch64 machine code, including the arm64e pointer-authentication instructions |
| `devicetree` | Apple's flattened device tree, the description of the hardware |
| `plistdump` | BuildManifest.plist and other property lists, with firmware digest checks |

## What it cannot do

- **Encrypted parts stay unreadable.** The AEA-encrypted disk images, including the main iOS file system, and the Secure Enclave firmware are encrypted with keys this project does not have. XZemulator does not implement decryption. In the iOS 27.0.1 IPSW above, that is 93.6% of the bytes. The readable parts are the kernelcache, the ramdisks, the device tree, and most coprocessor firmware.
- **It is not yet an emulator.** It cannot run iOS or iOS apps in this release.

## Legal

XZemulator contains no Apple firmware, code, or files taken from an IPSW. Bring your own IPSW and follow the laws and license terms that apply to you.

Apple, iOS and iPhone are trademarks of Apple Inc. XZemulator is an independent research project and is not affiliated with, endorsed by, or sponsored by Apple Inc.

XZemulator is released under the [MIT License](LICENSE). The license covers XZemulator's own code, including the 0.2.0 release. The bundled third-party components keep their own licenses: CPython, OpenSSL, libffi, NSIS and the AppImage runtime among them. The installers include the full list in `LICENSE_NOTICE`.

This repository publishes the release builds. The source code is not published here yet.
