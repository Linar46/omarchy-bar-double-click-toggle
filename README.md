# Bar Double-Click Toggle + Bar Auto-Hide

An Omarchy full-bar plugin that makes the stock bar's double-left-click background cycle opt-in and adds Windows-style auto-hide.

The stock bar cycles its background from **opaque → glassy → clear** when you double-left-click empty center-bar space. This plugin keeps the stock bar, widgets, layout, and drag-to-move gesture, but ignores the gesture unless you enable it. The bar can also slide away to its screen edge and return when the pointer reaches that edge.

## Install

```bash
omarchy plugin add https://github.com/Linar46/omarchy-bar-double-click-toggle.git --enable
```

This is a `bar` plugin, so it replaces the active bar. Your existing `bar.layout`, position, and widget settings in `~/.config/omarchy/shell.json` are preserved. The plugin id stays `io.github.linar46.bar-double-click-toggle` so existing `shell.json` entries and updates keep working. If the new bar is not picked up immediately, run `omarchy restart shell` once.

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

## Auto-hide (Windows-style)

The bar can slide away and reveal itself when the pointer reaches its screen edge, like the Windows taskbar. It is **off by default**.

### shell.json configuration

Enable it in `~/.config/omarchy/shell.json` by adding these keys to the existing `bar` object. Keep the `id`, `position`, `background`, and `layout` that are already there:

```json
{
  "bar": {
    "id": "io.github.linar46.bar-double-click-toggle",
    "position": "right",
    "autohide": true,
    "autohideDelay": 500,
    "autohideRevealDelay": 0,
    "autohidePeek": 2
  }
}
```

| Key | Default | Range | Meaning |
|---|---|---|---|
| `autohide` | `false` | `true` / `false` | Turn auto-hide on |
| `autohideDelay` | `600` | `0`–`5000` | ms after the pointer leaves before the bar hides |
| `autohideRevealDelay` | `120` | `0`–`5000` | ms of hovering the edge before the bar shows |
| `autohidePeek` | `2` | `1`–`12` | px of the bar kept on screen as the invisible hover target |

The `bar` keys are read by this plugin; `position` can be `top`, `bottom`, `left`, or `right` and the trigger edge follows it. The file hot-reloads on save, so no shell restart is needed. To turn auto-hide off again, set `"autohide": false`; the rest of the bar stays exactly as it is.

While auto-hide is on, the bar reserves no screen space, so windows use the whole workspace; the revealed bar slides in over them and slides away again when the pointer leaves. It stays out while a bar panel, tooltip, widget drag, or bar move is open, and the parked strip is invisible rather than a painted line at the edge. `Super + Shift + Space` / Hide Bar still wins over auto-hide: a manually hidden bar stays hidden.

## How it works

This plugin contains a copy of Omarchy's bar engine (`Bar.qml` and `BarModel.js`) from Omarchy 4.5.0. The double-click change is in `CenterGestureArea.onDoubleClicked`:

```qml
if (mouse.button === Qt.LeftButton && root.doubleClickCyclesBackground) {
  root.cycleBackground()
  mouse.accepted = true
}
```

`doubleClickCyclesBackground` follows the `bar-double-click-background` toggle flag. It reuses the same toggles-directory `FileView` that already watches Omarchy's `bar-off` flag, so the setting updates live as the flag is created or removed.

Auto-hide lives in `BarPanel`: `pointerInside` tracks the bar's hover, `revealed` drives the margin animation, and `activePopout` / `tooltipShown` / drag state hold the bar out while something needs it. The parked surface keeps a `autohidePeek`-pixel input strip so hover works without unmapping layer surfaces.

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

and port any relevant upstream changes. The local delta is intentionally small: the double-click toggle (one property, one toggle probe, one shared directory watch, one condition in the double-click handler) plus an opt-in auto-hide block in `BarPanel` and its config keys in `applyBarConfig`.

## Privacy and security

The plugin makes no network requests, collects no data, and does not rewrite your shell configuration. It reads one local toggle flag and renders the bar. Like any `bar` plugin, installing it with `--enable` selects it as the active Omarchy bar, which is the expected and explicit effect of enabling it.

## License

MIT. The bar engine is derived from Omarchy's MIT-licensed shell, copyright (c) David Heinemeier Hansson. See [LICENSE](LICENSE) and [NOTICE](NOTICE).
