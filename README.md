# SigRoam firmware (Scout Lite)

Closed-source **scanner firmware** for the PINGEQUA **Scout Lite** board
(ESP32-C5 + GPS + microSD). This repository publishes the production
**app image**. Source is not here.

This is **not** v1.0. A download is not a claim that the device is
field-ready.

Product name: **SigRoam**. Board name: **Scout Lite**. Do not write
"SigRoam Lite".

## Download

[Release `app-249eda5e`](https://github.com/pingequalab/sigroam-firmware/releases/tag/app-249eda5e)
— `sigroam_lite.bin` (1398400 B).

SHA-256: `249eda5e681368a57fb8d998e9926fe10875af2077bcbc6a9e30a68554adf502`

## Pair with FAP v0.4

Companion app: [SigRoam Wardriving `v0.4`](https://github.com/pingequalab/sigroam-wardriving/releases/tag/v0.4)
(`sigroam-0.4.fap`, SHA-256 `74e2f11a0543bd482c1d0539954588df1fb177d8f16ecc65bc14d573c44c219b`).

The eight-byte wire handshake is still `Firmware: Marauder`. Do not mix
this image with a different FAP, or the Official FAP with a different
image.

Factory boards ship **ESP32 Marauder**. Flashing this file is optional.

## Flash

Write **only** the app at **`0x20000`**.

Do **not** use the Marauder C5 layout (app at `0x10000`). Do **not**
flash this file from
[flash.pingequa.com/devices/scout-lite](https://flash.pingequa.com/devices/scout-lite)
— that page recovers Marauder at `0x10000`.

Unplug the Flipper before USB-C flashing. Unplug Scout USB-C before
using the board on Flipper 5 V. Those two setups cannot run together.

Receive-only: no deauth, handshake capture, evil twin, SoftAP, or
`esp_wifi_80211_tx`.

## License

MIT. See [LICENSE](LICENSE).

Finished boards are intentional radiators under FCC Part 15 / SDoC.
Educational and lawful network research only.
