# Skyward 0.3.1 — piano / waffle

Unofficial Skyward port for:

- Xiaomi Pad 8 Pro (`piano`), HyperOS 4 `OS4.0.6.0.XPYCNXM`
- OnePlus 12 (`waffle`), ColorOS 17 / Android 17 `PJD110_17.0.0.100(CN01)`

## Images

| Device | Image | SHA-256 |
| --- | --- | --- |
| piano | `skyward-0.3.1-piano-OS4.0.6.0.XPYCNXM.img` | `d29a3da1b0c657464a0eac02e14c247dc7e88ddfbccff0f150b8384b97c35974` |
| waffle | `skyward-0.3.1-waffle-PJD110_17.0.0.100.img` | `329dbe58f1ba57e79f06941564dea58da9d404018a85a02a44d4d6a1205ac255` |

Flash each to both recovery slots:

```sh
fastboot flash recovery_a <image>
fastboot flash recovery_b <image>
fastboot reboot recovery
```

The images are attached to the GitHub release rather than committed: each is
about 100 MiB and GitHub rejects files over 100 MB in a repository.

## Build

See `README.md` and `docs/PORTING.md`. The stock recovery images are not
included; get them from the matching full OTA and pass one to
`scripts/build.sh`.

## Verified on hardware

- piano: boots, 144 Hz display mode, touch, portrait/landscape switch,
  decryption (`/data`, `/sdcard`), Wi-Fi, battery.
- waffle: boots, 120 Hz display mode, touch, Wi-Fi (about 70 s to appear),
  `/data`, battery, haptics including touch taps; modem/MSS stays off to
  avoid an EDL reset.
