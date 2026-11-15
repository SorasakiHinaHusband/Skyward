# Skyward for Xiaomi Pad 8 Pro (piano) and OnePlus 12 (waffle)

Two device ports of [kirklin/skyward](https://github.com/kirklin/skyward), a
Rust recovery that runs the phone's own drivers and services from the stock
firmware.

This tree is based on Skyward **0.3.1** plus the OnePlus compatibility work
published in [shawude/skyward-oneplus13-sun](https://github.com/shawude/skyward-oneplus13-sun)
(GPL-3.0-or-later). Two new device configurations were written from the
available device trees and from the devices' own stock firmware:

| Device | Codename | Target OS | Port status |
| --- | --- | --- | --- |
| Xiaomi Pad 8 Pro | `piano` | HyperOS 4 `OS4.0.6.0.XPYCNXM` | booted; display, touch, decryption, Wi-Fi and the landscape/portrait switch verified |
| OnePlus 12 | `waffle` | ColorOS 17 / Android 17, build `PJD110_17.0.0.100(CN01)` (device currently on `PJD110_17.0.1.202(Milk！)`) | display at 120 Hz, touch, Wi-Fi, data, battery, haptics (including touch taps) and screenshots verified; modem not started |

Both images were flashed and checked on the two devices.

## Download

The two images are attached to the
[**v0.3.1-piano-waffle release**](https://github.com/SorasakiHinaHusband/Skyward/releases/tag/v0.3.1-piano-waffle):

| Image | Base | SHA-256 |
| --- | --- | --- |
| `skyward-0.3.1-piano-OS4.0.6.0.XPYCNXM.img` | piano stock recovery from HyperOS 4 `OS4.0.6.0.XPYCNXM` | `d29a3da1b0c657464a0eac02e14c247dc7e88ddfbccff0f150b8384b97c35974` |
| `skyward-0.3.1-waffle-PJD110_17.0.0.100.img` | waffle stock recovery from ColorOS 17 `PJD110_17.0.0.100(CN01)` H.11 | `329dbe58f1ba57e79f06941564dea58da9d404018a85a02a44d4d6a1205ac255` |

They are release assets rather than committed files, because each is about
100 MiB. The stock recovery images used as bases for rollback are not
committed either; get them from the matching full OTA (`docs/PORTING.md`).

### Waffle base

The official ColorOS 17 full package is `PJD110_17.0.0.100(CN01)` H.11
(8,853,874,090 bytes). Its recovery partition is 100 MiB and its
`init.rc` still starts OnePlus's `phoenix_recovery`, which the packer
rewrites to start Skyward's `recovery` service instead. The connected
phone currently runs the newer `PJD110_17.0.1.202(Milk！)` build; Skyward
reads the installed system's properties and services at runtime, so the
same image serves both.

## Flashing (to be done by the user)

Only the recovery partition is touched; `boot`, `init_boot`, `vendor_boot`,
`dtbo` and `vbmeta` stay as they are. Flash both slots so a slot switch does
not lose Skyward:

```sh
# piano
fastboot flash recovery_a out/skyward-0.3.1-piano-OS4.0.6.0.XPYCNXM.img
fastboot flash recovery_b out/skyward-0.3.1-piano-OS4.0.6.0.XPYCNXM.img

# waffle
fastboot flash recovery_a out/skyward-0.3.1-waffle-PJD110_17.0.0.100.img
fastboot flash recovery_b out/skyward-0.3.1-waffle-PJD110_17.0.0.100.img

fastboot reboot recovery
```

See [`docs/TESTING.md`](docs/TESTING.md) for the first-boot checklist and
log capture.

Rollback to stock uses the matching file from `stock/`:

```sh
fastboot flash recovery_a stock/piano/recovery.img
fastboot flash recovery_b stock/piano/recovery.img
fastboot reboot recovery
```

(Use `stock/waffle/recovery.img` for the OnePlus ColorOS 17 base; `stock/waffle16/recovery.img` is the older ColorOS 16 base.)

## Building

Requirements:

- Rust 1.100.0 with the `aarch64-linux-android` target
- Android NDK r29
- `cargo`, `git`, `curl`

```sh
export ANDROID_NDK_HOME=$HOME/Android/Sdk/ndk/29.0.14206865
scripts/build.sh piano stock/piano/recovery.img
scripts/build.sh waffle stock/waffle/recovery.img
```

`scripts/fetch-stock-piano.sh` re-downloads the piano stock recovery from
Xiaomi's OTA by HTTP range and checks its SHA-256. For waffle, supply a full
OTA zip and extract the recovery partition:

```sh
scripts/extract-ota.sh PJD110-full-ota.zip stock/waffle recovery
```

## Layout

```
skyward/                 Skyward 0.3.1 + OnePlus fixes + piano/waffle configs
  devices/piano.toml
  devices/waffle.toml
scripts/                 build, stock fetch, OTA extract
tools/ota-extract/       small Rust tool that reads payload.bin in a zip
patches/                 diff of this tree against upstream Skyward
docs/PORTING.md          hardware analysis and configuration decisions
docs/TESTING.md          first-boot checklist and log capture
out/                     built recovery images
stock/                   stock recovery images for rollback
```

## Known limitations

- `waffle` was booted and exercised on the connected phone (`PJD110_17.0.1.202`):
  display, touch, key chord, screenshots, `/data` and `/sdcard` mounts, and
  Wi-Fi (DHCP address and traffic) all work. `piano` has not been booted.
- `waffle`: the cellular modem (`mss`) is deliberately not started. Starting
  it on this "Milk" kernel resets the phone to EDL within about a minute;
  Wi-Fi works without it, and recovery does not need cellular. EFS/IMEI
  partitions are still readable for backup.
- `piano`: the panel is portrait while the UI's DRM mode is landscape, so
  the image swaps the touch axes and inverts Y (`[touch] swap_xy`,
  `invert_y`), matching TWRP's flags.
- `piano`: **Settings → 屏幕方向** switches between landscape and portrait;
  the choice is written to the settings and the recovery restarts to apply
  it. In portrait the pages are turned a quarter counter-clockwise and the
  touch grid is used directly. Both orientations were checked on the tablet.
- `piano`: decryption works (Xiaomi's generic weaver is started directly);
  the UI shows `已解密`.
- `piano`: the refresh-mode fix (fastest mode by name) is built into the
  current image but has not been checked on the tablet yet.
- `piano`: decryption depends on Xiaomi's HyperOS 4 secure stack and has not
  been exercised. `touch_report_debug` (the stock touch-processing launcher)
  is started from `/odm/bin`; touch is expected to work once the ODM partition
  is mounted.
- `waffle`: MTP is disabled (`mtp = false`); OnePlus's recovery USB gadget
  resets the port when adb and MTP are offered together.
- `waffle`: haptics use `qcom-hv-haptics`; touch taps, the back gesture and
  the strength slider were all verified. The driver leaves
  `lra_frequency_hz` at 0 until the motor is first played, so the recovery
  derives the frequency from `t_lra_us`.
- `waffle`: the panel is switched to its fastest mode, `1440x3168x120cmd`
  (120 Hz); the mode's kernel `vrefresh` is not trusted, the number at the
  end of its name is.
- `waffle`: Wi-Fi takes about 70 s to appear in recovery: CNSS waits for the
  modem's DMS service until its retries run out and then uses a default MAC.
  Cellular/modem stays off on purpose.
- Both: the first start may take 30–60 s while the data partition and key
  services come up.

## License

Skyward is GPL-3.0-or-later; this port keeps that license. See
[`skyward/LICENSE`](skyward/LICENSE).
