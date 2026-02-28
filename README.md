# SC HOTAS Mapping Helper

A browser-based tool for testing and creating HOTAS binding profiles for [Star Citizen](https://robertsspaceindustries.com/) — built specifically for **Linux** players who don't have access to tools like Thrustmaster's T.A.R.G.E.T. software.

Works with **any joystick or throttle** recognized by the browser Gamepad API. Includes a built-in device database with profiles for **10 popular HOTAS devices** (T16000M, TWCS, VKB Gladiator, VirPil, Logitech, Saitek, CH Products, and more) with automatic detection and correct axis mapping.

> **Zero install. No dependencies. Just open the HTML file in your browser.**

---

## What It Does

### Test Mode

Plug in your HOTAS and see every input visualized in real-time:

- **Crosshair displays** for stick and ministick axes
- **Axis bars** with deadzone indicators and live values
- **Button grid** that lights up on press, showing the mapped SC action
- **Hat switch** directional visualization
- **Live action log** capturing every input with timestamps

Useful for verifying that your devices are detected correctly on Linux, checking axis ranges, identifying button indices, and spotting stick drift before you even launch the game.

### Bind Mode

Create a full Star Citizen keybinding profile from scratch — or import and edit an existing one:

1. Open a category (Flight Movement, Weapons, Shields, Mining, Salvage, etc.)
2. Click **Bind** next to any Star Citizen action
3. Press a button or move an axis on your HOTAS
4. The input is captured and mapped
5. Hit **Export XML** when done — ready to drop into your SC install

Features:

- **455 Star Citizen actions** across 23 categories, sourced from Boxxy-Binder's AllBinds.xml — covering flight, combat, targeting, mining, salvage, scanning, EVA, ground vehicles, turrets, tractor beam, docking, and more
- **Tier filter** — show Essential actions only (~56), Essential + Extended (~315), or all 455. Keeps the UI manageable while giving access to everything
- **Search & filter toolbar** — text search, type filter (axes/buttons), status filter (bound/unbound/conflicts), and tier filter
- **Device auto-detection** — database of 10 HOTAS devices with correct SC axis name mappings (including proper `rotz` for twist axes)
- **Three-tier axis mapping** — device-specific profiles → generic HID fallback → raw index, so axes map correctly across different hardware
- **Persistent bindings** — bindings, inversions, and profile name saved to localStorage (survives page refresh)
- **Axis inversion toggles** — per-action inversion with proper `optionGroup`-based XML export matching Star Citizen's format
- **Conflict highlighting** — duplicate bindings are visually highlighted in red
- **Friendly input names** — `js1_button3` displays as `JS1 Btn 3`, `js2_rotz` as `JS2 Rz Axis`
- **Live input monitor** sidebar showing all axes and buttons in real-time while binding
- **Import XML** to load and edit existing profiles (with full inversion state round-trip)
- **Export** as a properly formatted SC actionmap XML
- **Copy to clipboard** or **download** the XML file directly

---

## Supported Devices

The tool includes profiles for these devices (auto-detected by name):

| Device | Type | Axes Mapped |
|--------|------|-------------|
| Thrustmaster T.16000M FCS | Joystick | X, Y, Rz (twist) |
| Thrustmaster TWCS Throttle | Throttle | X, Y, Z (slider), Rx (rocker) |
| VKB Gladiator NXT / EVO | Joystick | X, Y, Rz (twist) |
| VirPil Controls | Joystick | X, Y, Rz |
| Logitech Extreme 3D Pro | Joystick | X, Y, Rz, Slider |
| Logitech X52 / X52 Pro | HOTAS | X, Y, Rz |
| Logitech X56 Rhino | HOTAS | X, Y, Rz |
| Saitek / Mad Catz | Various | X, Y, Rz |
| CH Products | Various | X, Y, Rz |

Unrecognized devices use a generic fallback mapping. You can always verify your axis mappings in Test Mode.

---

## Getting Started

### Quick Start

1. Download `hotas_tester.html`
2. Open it in **Firefox** or **Chrome**
3. Plug in your HOTAS (joystick first, then throttle, for consistent ordering)
4. Press any button — the tool auto-detects your devices

### Installing a Generated Profile in Star Citizen

1. Copy your exported XML file to:
   ```
   <SC Install>/StarCitizen/LIVE/USER/Client/0/Controls/Mappings/
   ```
   Create the `Mappings` folder if it doesn't exist.

2. Launch Star Citizen, open the console with `~` and type:
   ```
   pp_RebindKeys layout_YourProfileName_exported
   ```
   (without the `.xml` extension)

3. If your joystick and throttle are swapped (JS1/JS2 reversed), run:
   ```
   pp_resortdevices joystick 1 2
   ```

### Linux Notes

- The tool uses the [Gamepad API](https://developer.mozilla.org/en-US/docs/Web/API/Gamepad_API), which works out of the box on Firefox and Chrome on Linux.
- Your devices should be visible at `/dev/input/js*`. If not, check that your user is in the `input` group.
- Plug in the joystick **before** the throttle for consistent `js1`/`js2` ordering — Linux doesn't always assign them the same way.
- Hat switches are detected both as axes (common on Linux HID drivers) and as buttons, with automatic fallback.
- Axis indices may differ from what Star Citizen sees through Wine/Proton. The tool lets you see the raw indices so you can verify.

---

## Troubleshooting (Linux / Wine / Proton)

### TWCS Throttle shows up as a second "T.16000M"

This is the most common issue on Linux. Wine's HID layer reports both the joystick and the TWCS throttle with the same product name `T.16000M`. Star Citizen then shows two identical "Joystick" entries in the keybinding UI instead of recognizing one as a throttle.

**What still works:** All button and axis bindings using `js1_` / `js2_` prefixes work correctly because they reference the device by instance number, not by name. Your flight controls, weapons, and throttle should all respond.

**What breaks:** The `<deviceoptions>` XML section matches devices by name, so it's impossible to set different deadzones for the joystick vs. the throttle in the XML alone.

**Fix:** After loading the profile with `pp_RebindKeys`, go to Options > Keybinding > Advanced Controls and adjust deadzones per-device in the UI. The ministick on the TWCS typically needs a higher deadzone (~15%) to avoid drift, while the main joystick axes can be lower (~5%).

### JS1 and JS2 are swapped

If the throttle is responding to joystick bindings (or vice versa), your devices were enumerated in the wrong order. Fix it with the console command:

```
pp_resortdevices joystick 1 2
```

Then re-import the profile. To prevent this, try plugging in the **joystick first**, then the throttle. On some Linux setups, USB port order determines device numbering.

### No gamepad detected in the browser tester

Make sure your user is in the `input` group:

```bash
sudo usermod -aG input $USER
```

Then log out and back in. Verify the devices appear at `/dev/input/js0` and `/dev/input/js1`.

Firefox may also need `dom.gamepad.extensions.enabled` set to `true` in `about:config`.

### Axes respond but values seem wrong

Star Citizen through Wine/Proton may see different axis indices than the browser Gamepad API. Use the HOTAS Tester's Test Mode to identify which physical axis maps to which index in the browser, then cross-reference with the in-game binding screen to confirm they match. If they differ, use Bind Mode to create a profile that maps the correct browser-detected indices.

---

## How It Was Built

This tool was co-created with [Claude](https://claude.ai) (Anthropic) using [Claude Code](https://docs.anthropic.com/en/docs/claude-code). The action database (455 actions across 23 categories) was extracted from [Boxxy-Binder](https://github.com/BoxximusPrime/Boxxy-Binder)'s AllBinds.xml, with tier classification and `optionGroup` metadata for correct SC XML export.

---

## Contributing

Contributions are welcome! Some ideas:

- **More pre-made profiles** for other HOTAS setups (X52, X56, VKB, Virpil, etc.)
- **Dual-stick (HOSAS) support** with dedicated layout templates
- **Dynamic Test Mode UI** generated from device profiles instead of hardcoded panels
- **Deadzone/curve editor** with visual preview
- **Improved axis auto-detection** for more Linux HID driver variants

If you create a profile for your own HOTAS and want to share it, PRs are very welcome.

---

## License

This project is licensed under the **GNU General Public License v3.0** — see [LICENSE](LICENSE) for details.

---

## Links

- [Star Citizen](https://robertsspaceindustries.com/)
- [Boxxy-Binder](https://github.com/BoxximusPrime/Boxxy-Binder) — the Tauri/Rust SC binding tool that inspired many improvements here
- [RSI Knowledge Base: Custom Profiles](https://support.robertsspaceindustries.com/hc/en-us/articles/360000183328)
- [Star Citizen Wiki: Controls](https://starcitizen.fandom.com/wiki/Controls)
