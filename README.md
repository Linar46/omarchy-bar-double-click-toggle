# Bar Double-Click Toggle

An Omarchy full-bar plugin that makes the stock bar's double-left-click background cycle **opt-in**.

Stock Omarchy cycles the bar background from **opaque → glassy → clear** when you double-left-click empty center-bar space. If that keeps happening by accident, this plugin keeps the stock bar, widgets, layout, and drag-to-move gesture, but ignores the double-click unless you enable it.

## Install

```bash
omarchy plugin add https://github.com/Linar46/omarchy-bar-double-click-toggle.git --enable
```

This is a `bar` plugin, so it replaces the active bar. Your existing `bar.layout`, position, and widget settings in `~/.config/omarchy/shell.json` are preserved. If the new bar is not picked up immediately, run `omarchy restart shell` once.

## Toggle the gesture

The gesture is **off by default**. Enable or disable it whenever you want:

```bash
omarchy toggle bar-double-click-background on    # allow double-click background cycling
omarchy toggle bar-double-click-background off   # disable it
```

Check the state with:

```bash
omarchy toggle enabled bar-double-click-background
# exit code 0 = on, 1 = off
```

The change takes effect immediately, with no shell restart. The toggle flag lives at:

```text
~/.local/state/omarchy/toggles/bar-double-click-background
```

The background remains available without the gesture through **Appearance → Bar Background**, or:

```bash
omarchy bar background opaque
omarchy bar background glassy
omarchy bar background none
omarchy bar background cycle
```

## How it works

This plugin contains a copy of Omarchy's bar engine (`Bar.qml` and `BarModel.js`) from Omarchy 4.5.0. The only behavioral change is in `CenterGestureArea.onDoubleClicked`:

```qml
if (mouse.button === Qt.LeftButton && root.doubleClickCyclesBackground) {
  root.cycleBackground()
  mouse.accepted = true
}
```

`doubleClickCyclesBackground` follows the `bar-double-click-background` toggle flag. It reuses the same toggles-directory `FileView` that already watches Omarchy's `bar-off` flag, so the setting updates live as the flag is created or removed.

First-party widget files are not bundled. The bar resolves enabled widgets through Omarchy's plugin registry, so copies of `widgets/` and `indicators/` inside a third-party full-bar plugin would be inert.

## Requirements

- Omarchy Quattro 4.5.0 or later (built against 4.5.0-3).
- The standard Omarchy Quickshell bar engine and `omarchy` CLI.

## Remove

Return to the built-in bar and restore the stock always-on gesture:

```bash
omarchy bar reset
omarchy plugin remove io.github.linar46.bar-double-click-toggle --yes
```

`omarchy bar reset` alone is enough to switch back without deleting the plugin.

## Maintenance

A full-bar plugin carries a copy of Omarchy's bar engine, so upstream bar fixes do not flow into it automatically. After an Omarchy update, compare `Bar.qml` and `BarModel.js` with:

```text
/usr/share/omarchy/shell/plugins/bar/
```

and port any relevant upstream changes. The local delta is intentionally small: one property, one toggle probe, one shared directory watch, and one condition in the double-click handler.

## Privacy and security

The plugin makes no network requests, collects no data, and does not rewrite your shell configuration. It reads one local toggle flag and renders the bar. Like any `bar` plugin, installing it with `--enable` selects it as the active Omarchy bar, which is the expected and explicit effect of enabling it.

## License

MIT. The bar engine is derived from Omarchy's MIT-licensed shell, copyright (c) David Heinemeier Hansson. See [LICENSE](LICENSE) and [NOTICE](NOTICE).
