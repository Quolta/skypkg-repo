# skypkg Repository

Arch-style package repository for SkyOS 5.0.

## Usage

From inside SkyOS:
```bash
skyjs skypkg.js -Sy          # sync database
skyjs skypkg.js -S busybox   # install busybox
```

## Repository Structure

```
core/
  core.db.tar.gz              # package database
  busybox-1.36.1-1-i586.pkg.tar.gz
```

## About

SkyOS is a proprietary x86 operating system from 2007 by Robert Szeleney.
This repository provides cross-compiled packages via `skypkg`, an Arch Linux
pacman-compatible package manager written in JavaScript running on the SkyJS runtime.

All packages are ELF 32-bit i386 binaries linked against SkyOS's `libsky.so`.
