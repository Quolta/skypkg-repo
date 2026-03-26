# Contributing to skypkg-repo

## Adding New Packages

### Prerequisites
- Linux host (tested on CachyOS/Arch, should work on any x86_64 Linux)
- GCC with multilib support (`gcc-multilib` or equivalent)
- SkyOS cross-compilation sysroot (contact maintainers)

### Package Creation Steps

1. **Cross-compile your software:**
   ```bash
   export CC="gcc"
   export CFLAGS="-m32 -march=i586 -nostdinc ..."  # See README for full flags
   # Configure and build as appropriate for your software
   ```

2. **Create package directory:**
   ```bash
   mkdir -p pkg/usr/bin
   cp your-binary pkg/usr/bin/
   ```

3. **Write .PKGINFO:**
   ```
   pkgname = your-package
   pkgver = 1.0.0
   pkgrel = 1
   pkgdesc = Brief description
   arch = i586
   depend = libsky
   ```

4. **Build package:**
   ```bash
   cd pkg && tar czf ../your-package-1.0.0-1-i586.pkg.tar.gz .PKGINFO usr/
   ```

5. **Test locally** by copying to QEMU FAT drive and installing with `skypkg -U`.

6. **Submit PR** with the `.pkg.tar.gz` file in the `core/` directory.

### Important Notes

- **NEVER define `__GLIBC__`** in headers — it breaks binary compatibility with SkyOS's newlib
- **No symlinks** — SkyOS LiveCD doesn't support them. Use `cp` in .INSTALL scripts.
- **Test on SkyOS** — always verify packages work in QEMU before submitting
- **Keep JS files under 6KB** — QuickJS has memory limits on SkyOS
