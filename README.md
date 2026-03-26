# skypkg Repository — Package Repository for SkyOS 5.0

Arch-style package repository for SkyOS 5.0, a proprietary x86 operating system
from 2007 by Robert Szeleney. Packages are cross-compiled on Linux (CachyOS/Arch)
and served via GitHub Pages over HTTPS.

## Quick Start (Inside SkyOS QEMU)

```bash
# Bootstrap (first boot only)
mount /dev/hd1 /fat0 -t fat
cp /fat0/busybox /tmp/busybox
chmod +x /tmp/busybox
export PATH=/tmp:$PATH
busybox mkdir -p /var/lib/skypkg/local /var/lib/skypkg/sync /var/lib/skypkg/cache /var/log /etc
busybox cp /fat0/skypkg.conf /etc/skypkg.conf

# Sync and install from remote repo
/fat0/skyjs /fat0/skypkg-lite.js -Sy
/fat0/skyjs /fat0/skypkg-lite.js -S busybox
/fat0/skyjs /fat0/skypkg-lite.js -S skyjs
/fat0/skyjs /fat0/skypkg-lite.js -S skypkg

# Fix: SkyOS LiveCD doesn't support symlinks
busybox cp /usr/bin/busybox /usr/bin/sh

# Now skypkg is self-hosting!
/usr/bin/skypkg -Q
/usr/bin/skypkg -Sy
/usr/bin/skypkg -Ss <query>
/usr/bin/skypkg -S <package>
```

## Available Packages

| Package | Version | Size | Description |
|---------|---------|------|-------------|
| busybox | 1.36.1-1 | 376KB | Core Unix utilities: tar, gzip, sh, ls, cat, cp, mv, rm, mkdir, chmod, grep, sed, awk, vi, find, sort, head, tail, wc, xargs, sha256sum, and 100+ more |
| skyjs | 1.0.0-1 | 1.9MB | QuickJS + OpenSSL 1.1.1w + libuv 1.48.0 JavaScript runtime. Provides HTTPS client, file I/O, and shell execution. |
| skypkg | 0.1.0-1 | 3.6KB | Self-hosting Arch-style package manager. Wrapper script + config + JS implementation. |

## Repository Structure

```
core/
  core.db.tar.gz                        # Package database (Arch-compatible format)
  busybox-1.36.1-1-i586.pkg.tar.gz     # Package files
  skyjs-1.0.0-1-i586.pkg.tar.gz
  skypkg-0.1.0-1-any.pkg.tar.gz
```

## Architecture

- **Target:** SkyOS 5.0 (ELF 32-bit i386, `libsky.so` C library)
- **Host build system:** CachyOS (Arch Linux), GCC 15.2.1
- **Cross-CFLAGS:** `-m32 -march=i586 -nostdinc -D__SKYOS__ -DSKYOS`
- **Package format:** Arch-compatible `.pkg.tar.gz` with `.PKGINFO`, `.INSTALL`, `.FILELIST`
- **Database format:** Arch-compatible `desc` files with `%NAME%`, `%VERSION%`, etc.
- **Transport:** HTTPS via GitHub Pages, TLS 1.2

## For Contributors

### How Packages Are Built

All packages are cross-compiled from Linux targeting SkyOS's i586 ABI:

```bash
# Standard SkyOS cross-compilation flags
SKYOS_CFLAGS="-m32 -march=i586 -nostdinc \
  -isystem /usr/lib/gcc/x86_64-pc-linux-gnu/15.2.1/include \
  -I$HOME/skyos-cross/sysroot/usr/include \
  -D__SKYOS__ -DSKYOS -D_GNU_SOURCE -D_POSIX_C_SOURCE=200112L \
  -Ulinux -U__linux -U__linux__ -U__gnu_linux__ \
  -fno-builtin -fno-stack-protector -fpic -O2"
```

### How to Add a Package

1. Cross-compile the software with SkyOS CFLAGS
2. Create a `.PKGINFO` file with package metadata
3. Create a `.pkg.tar.gz` containing `.PKGINFO` + installed files
4. Run: `python3 skypkg-repo-add core.db.tar.gz your-package.pkg.tar.gz`
5. Commit and push — GitHub Pages serves it automatically

### Known Limitations

- **No symlinks on LiveCD** — SkyOS LiveCD filesystem doesn't support `ln -s`. Use `cp` instead.
- **JS file size limit ~6KB** — QuickJS on SkyOS has a practical limit due to memory constraints. Keep scripts compact.
- **MAX_TASKS = 20** — SkyOS supports only 20 concurrent processes.
- **Default stack = 1MB** — but OpenSSL + QuickJS are memory-hungry.
- **No /proc filesystem** — many Linux utilities won't work (ps, top, free, etc.)
- **`closesocket()` not `close()`** — SkyOS uses Windows-style socket closing.
- **Packages lost on reboot** — LiveCD boots to RAM. Need installed SkyOS for persistence.

### SkyOS Sysroot

The cross-compilation sysroot contains headers and libraries extracted from the
SkyOS 5.0 retail ISO. It includes the original SDK headers (newlib-derived) plus
compatibility stubs added during the porting process.

## Technical Background

SkyOS was a proprietary operating system created by Robert Szeleney (Slovakia),
developed from ~2001-2009. It featured a custom kernel, SkyGI graphical toolkit,
and partial POSIX compatibility. The retail version (5.0.6947) was the last release.
Source code was never published.

This project reverse-engineered the kernel's syscall table (362 entries, 293 implemented),
mapped the `libsky.so` API surface (2,192 exported symbols), and built a complete
cross-compilation toolchain to produce new software for the OS.

## License

Repository tooling: MIT. Individual packages retain their upstream licenses.
