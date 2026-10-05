# Native Arch package

This recipe builds the current repository checkout, including local changes,
and packages the native Tauri executable with Arch's GTK, WebKitGTK and GStreamer
libraries. Run it from `packaging/arch` in an x86_64 Arch checkout.

## Build and install

Install `base-devel` and the recipe's runtime/build dependencies first.
`makepkg -s` can install missing dependencies when you authorize it.

```sh
cd packaging/arch
makepkg
sudo pacman -U ./blinkeye-2.8.1-1-x86_64.pkg.tar.zst
blinkeye
```

Set `TMPDIR` to a disk-backed scratch directory before building on machines
with a small RAM-backed `/tmp`. To limit compilation memory, use
`CARGO_BUILD_JOBS=1 makepkg`. The recipe otherwise respects the caller's
temporary-directory and Cargo job configuration. Leave `CARGO_TARGET_DIR`
unset: packaging expects the executable in `src-tauri/target/release`.

The `Blink Eye (native)` desktop entry starts the `blinkeye` launcher.
Close any other Blink Eye instance before launching the installed package.
The package installs the application license, icons and reminder MP3 resources.

## Rendering defaults

The launcher defaults to X11/XWayland and disables WebKit's DMA-BUF renderer,
matching the original CachyOS/Niri compatibility setup. These are fallback
defaults, not a claim that every Arch desktop needs them. Both can be overridden:

```sh
GDK_BACKEND=wayland blinkeye
WEBKIT_DISABLE_DMABUF_RENDERER=0 blinkeye
```

Use `GDK_BACKEND=wayland blinkeye` for the native Wayland reminder presenter and
compositor-managed shortcut inhibition described in
[Linux reminder windows](../../docs/linux-wayland-reminders.md). The desktop
entry without that environment override uses X11 and does not exercise that path.

The raw `/usr/bin/Blink-Eye` executable and upstream-generated autostart entry
can bypass these launcher defaults. Review autostart separately on a desktop
that needs the rendering workaround.

## Packaging scope

`pkgver` must match the checked-out application version. This checkout-local
recipe is intended for repository builds; an AUR submission would need its own
versioned source/archive policy and naming approval. No AUR publication is part
of this contribution.

The license is `custom:Blink-Eye` because the upstream license includes commercial
restrictions in addition to GPLv3 references; the full `LICENSE` is installed.
Package-level LTO is disabled because GCC LTO objects in bundled SQLite caused
unresolved symbols in SQLx's Rust procedural-macro library during verification.
