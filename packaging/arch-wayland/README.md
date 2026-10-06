# Parallel native Wayland Arch package

This checkout-local source recipe builds `blinkeye-wayland` from the current
repository, including local runtime fixes. It can be installed alongside
`blinkeye`: it has no `provides` or `conflicts` entries and shares no package
files. Runtime and build dependencies, license handling and LTO settings match
`packaging/arch`.

## Build when runtime diagnosis is complete

Use an x86_64 Arch checkout with `base-devel` and the recipe's dependencies
already installed. From the repository root:

```sh
cd packaging/arch-wayland
TMPDIR="$HOME/.cache/agent-tmp" CARGO_BUILD_JOBS=1 makepkg
```

The scratch directory must already exist. Leave `CARGO_TARGET_DIR` unset:
packaging reads `src-tauri/target/release/Blink-Eye`. Do not reuse an existing
binary with `makepkg --repackage`: the Wayland config must be compiled into it.
`pkgver` must match the checkout's application version.

After separately authorizing installation, the package file is
`blinkeye-wayland-2.8.1-1-x86_64.pkg.tar.zst`. Start it with `blinkeye-wayland`
or the **Blink Eye (Wayland)** desktop entry.

## Rendering and profile isolation

The launcher leaves `GDK_BACKEND` unchanged. GTK chooses an available backend,
preferring Wayland before X11 in the tested GTK3 build. Explicit backend choices
and ordered lists remain GTK's responsibility. The package keeps its separate
Wayland edition name and profile even when an X11 fallback is selected.
The existing WebKit DMA-BUF compatibility default remains overridable:

```sh
WEBKIT_DISABLE_DMABUF_RENDERER=0 blinkeye-wayland
```

The Tauri build overlay sets identifier `com.blinkeye.wayland`, product name
`Blink Eye Wayland`, and main window title `Blink Eye (Wayland)`. It leaves the
main config untouched. Tauri merges objects but replaces arrays, so the overlay
retains the original main window settings when replacing its window array.

The Wayland profile uses `${XDG_DATA_HOME:-$HOME/.local/share}/com.blinkeye.wayland`
for databases and `${XDG_CONFIG_HOME:-$HOME/.config}/com.blinkeye.wayland` for
app configuration. The existing `com.blinkeye.app` profile is not migrated or
modified. Review autostart separately: the app's autostart plugin uses the
distinct product name but launches the raw executable, bypassing the launcher's
environment settings. The raw executable also bypasses them when started
directly.

## Installed paths and resource contract

- Launcher: `/usr/bin/blinkeye-wayland`
- Native executable: `/usr/bin/Blink-Eye-Wayland`
- Desktop entry: `/usr/share/applications/blinkeye-wayland.desktop`
- Icons: `/usr/share/icons/hicolor/{32x32,128x128,256x256}/apps/blinkeye-wayland.png`
- License: `/usr/share/licenses/blinkeye-wayland/LICENSE`
- Sounds: `/usr/lib/Blink Eye Wayland/{done.mp3,before_alert_sound.mp3}`

The locked Tauri 2.10.2 delegates `resource_dir()` to tauri-utils 2.8.2.
Its Linux implementation checks `../lib/<package_info.name>` beside the
executable, then `/usr/lib/<package_info.name>` outside an AppImage. Tauri
codegen obtains that name from `productName`, not the renamed executable.
Thus `/usr/lib/Blink Eye Wayland` is the required resource directory; renaming
the binary does not change it. The main config's sound resource mappings remain
unchanged.

## Source integrity and scope

To regenerate recipe-source checksums after editing the launcher, desktop entry
or overlay, run from the repository root:

```sh
TMPDIR="$HOME/.cache/agent-tmp" updpkgsums packaging/arch-wayland/PKGBUILD
(cd packaging/arch-wayland && TMPDIR="$HOME/.cache/agent-tmp" makepkg --verifysource)
```

The checksums cover these three recipe inputs, not the mutable repository
checkout. This is a local source-build recipe, not a versioned-source AUR
recipe. No AUR publication or packaging upstream PR is included.

The full upstream `LICENSE` is installed with `custom:Blink-Eye`, preserving
its commercial restrictions in addition to its GPLv3 references. No new
dependencies, assets or licenses are introduced.
