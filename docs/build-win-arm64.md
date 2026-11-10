# Building Rnote natively for Windows on ARM (aarch64)

Builds Rnote in the MSYS2 `CLANGARM64` environment. Upstream Rnote does not support this yet, so the build uses the
`arm64-upstream` branch of [sci42/rnote-win-arm64](https://github.com/sci42/rnote-win-arm64), which is upstream `main`
plus the build changes needed for aarch64.

## Required

- A Windows on ARM device (e.g. Snapdragon X)
- [MSYS2](https://www.msys2.org/), [rustup](https://rustup.rs/) and [Inno Setup 6](https://jrsoftware.org/isinfo.php)
  (only needed for the installer)
- Windows **Developer Mode** enabled (Settings → System → For developers). The repository contains symlinks
  (`crates/rnote-ui/po/zh_*.po`), which git can only create with it.

## Steps

1. Install the tools (PowerShell):
   ```powershell
   winget install MSYS2.MSYS2
   winget install Rustlang.Rustup
   winget install JRSoftware.InnoSetup
   rustup set default-host aarch64-pc-windows-gnullvm
   ```
2. Open **MSYS2 CLANGARM64** from the start menu (not UCRT64 or MSYS) and update it. Run this until it reports nothing
   to do, the window may close after the first run:
   ```bash
   pacman -Syu
   ```
3. Add cargo and Inno Setup to the `PATH` of the MSYS2 shell:
   ```bash
   echo 'export PATH="$PATH:$(cygpath "$USERPROFILE")/.cargo/bin:/c/Program Files (x86)/Inno Setup 6"' >> ~/.bashrc
   source ~/.bashrc
   ```
4. Clone the branch into a path **without spaces**:
   ```bash
   pacman -S --needed git just
   git clone -c core.symlinks=true -b arm64-upstream https://github.com/sci42/rnote-win-arm64.git /c/dev/rnote
   cd /c/dev/rnote
   ```
5. Install the build dependencies (GTK, libadwaita, meson, ...) and the Rust toolchain:
   ```bash
   just prerequisites-win-arm64
   ```
6. Configure and build the installer:
   ```bash
   just msys_env_prefix_path=C:/msys64/clangarm64 setup-win-installer rnote-win-installer-arm64
   just build-win-installer
   ```
   The installer ends up at `_mesonbuild/rnote-win-installer-arm64.exe`.

The first build takes a while, rebuilds after small changes are much faster.

## Running without the installer

Instead of step 6 you can install into the MSYS2 environment and run it from there:

```bash
just msys_env_prefix_path=C:/msys64/clangarm64 setup-win-installer
just build
just install
/c/msys64/clangarm64/bin/rnote.exe
```

`just install` ends with an error about `update-desktop-database`. That is a Linux tool and can be ignored, everything
else is installed at that point.
