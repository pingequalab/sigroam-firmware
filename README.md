<h1 align="center">SigRoam</h1>

<p align="center">
  <b>Receive-only dual-band wardriving firmware for Scout Lite.</b><br>
  The scanner on the ESP32-C5. The Flipper is the control head.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/radio-receive--only-brightgreen" alt="Receive-only">
  <img src="https://img.shields.io/badge/source-closed-lightgrey" alt="Closed source">
  <img src="https://img.shields.io/badge/status-not%20v1.0-yellow" alt="Not v1.0">
  <img src="https://img.shields.io/badge/board-Scout%20Lite-orange" alt="Scout Lite">
  <img src="https://img.shields.io/badge/companion-FAP%20v0.4-blue" alt="Companion FAP v0.4">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue" alt="License: MIT"></a>
</p>

---

**SigRoam** is dedicated scanner firmware for the PINGEQUA **Scout Lite** board
(ESP32-C5 + GPS + microSD). It enumerates 2.4 / 5 GHz Wi-Fi, observes BLE,
tags each record with a GNSS fix, and writes [WiGLE](https://wigle.net)-ready
CSV to the onboard card. The Flipper Zero does not scan: it starts and stops
the session and shows what the scanner reports.

This repository publishes the **production app image**. Source is not here.
The current tag is the first eight hex digits of that image’s SHA-256.

This is **not** product v1.0. A download is not a claim that the board is
field-ready.

Factory Scout Lite units ship **[ESP32 Marauder](https://github.com/justcallmekoko/ESP32Marauder)**.
Flashing SigRoam is optional. The Flipper companion
([SigRoam Wardriving v0.4](https://github.com/pingequalab/sigroam-wardriving/releases/tag/v0.4))
speaks the Marauder serial dialect either way.

| | |
|---|---|
| **Product** | SigRoam |
| **Board** | Scout Lite — not “SigRoam Lite” |
| **SoC** | ESP32-C5-WROOM-1U-N8R8 · 2.4 + 5 GHz |
| **GNSS** | Quectel L86-M33 (GPS / GLONASS / Galileo) |
| **Log** | microSD · WigleWifi CSV |
| **Host** | Flipper Zero · GPIO UART 13/14 · 5 V on pin 1 |
| **Wire handshake** | `Firmware: Marauder` (eight bytes, unchanged) |

## Why this firmware exists

Marauder, GhostESP, and Bruce are ESP32 wireless toolkits. Wardriving sits
next to deauth, handshake capture, and rogue AP. SigRoam is the other cut:
**one job, receive-only**, on Scout Lite’s C5 + GNSS + microSD, talking to a
Flipper with a 2 KB serial buffer.

| | **SigRoam** | **Marauder** | **GhostESP / Bruce** | **WiGLE app** |
|---|---|---|---|---|
| Job | Dual-band survey + GNSS log | Wi-Fi / BT toolkit | Red-team multi-tool | Phone wardrive |
| Attack TX | None | Deauth, handshake, rogue AP | Deauth, portal, spam | None |
| Radio | ESP32-C5 2.4 + 5 GHz | Many chips, incl. C5 | Board-dependent ESP32 | Phone Wi-Fi |
| GNSS + SD | Onboard Scout Lite | Optional add-ons | Board-dependent | Phone GNSS |
| Host | Flipper FAP | App / CLI / display | Flipper / WebUI / display | Phone |
| Source | Closed (binaries here) | Open | Open | Closed app |

Fetched 2026-09-18:
[Marauder](https://github.com/justcallmekoko/ESP32Marauder) (“offensive and defensive tools”);
C5 app offset `0x10000` — [wiki 2026-09-15](https://github.com/justcallmekoko/ESP32Marauder/wiki/update-firmware);
[GhostESP](https://github.com/GhostESP-Revival/GhostESP);
[Bruce](https://github.com/BruceDevices/firmware).
Laptop-class sniffing is still [Kismet](https://www.kismetwireless.net/).

Versus Marauder on the same board:

- **Passive scan** — no probe-request active scan, no `esp_wifi_80211_tx`
- **GPS-gated CSV** — a microSD row is committed only with a valid fix at
  observation time. Indoor no-fix does not write `0.0000000` (Null Island).
  That is design, not a fault
- **UART is the live view** — rate-limited for the Flipper RX buffer. The
  card is the record. A GPS query does not abort a running survey
- **Session sidecar** — CSV plus a manifest with drop counters
- **Same six host commands** — `info` · `wardrive` · `wardrive -serial` ·
  `stopscan` · `gpsdata` · `wardrivepoi`

## Release

| | |
|---|---|
| **Title** | [SigRoam 0.4 for Scout Lite](https://github.com/pingequalab/sigroam-firmware/releases/tag/app-249eda5e) |
| **Tag** | `app-249eda5e` (first eight hex of the image SHA-256 — **not** v1.0) |
| **File** | `sigroam_lite.bin` · **1398400** B |
| **SHA-256** | `249eda5e681368a57fb8d998e9926fe10875af2077bcbc6a9e30a68554adf502` |
| **Write** | App only at **`0x20000`** (`ota_0` on the SigRoam 8 MB layout) |
| **Companion** | [SigRoam Wardriving v0.4](https://github.com/pingequalab/sigroam-wardriving/releases/tag/v0.4) |
| **Factory** | ESP32 Marauder — leave it if you want the toolkit |
| **Not in this image** | Product v1.0 · on-device WiGLE HTTPS upload · merged first-install for stock Marauder |

Pairing with FAP v0.4 is verified. That is not “ready for the field.”

```text
shasum -a 256 sigroam_lite.bin
# 249eda5e681368a57fb8d998e9926fe10875af2077bcbc6a9e30a68554adf502
```

## Pair with FAP v0.4

| | |
|---|---|
| **App** | [sigroam-wardriving v0.4](https://github.com/pingequalab/sigroam-wardriving/releases/tag/v0.4) |
| **File** | `sigroam-0.4.fap` |
| **SHA-256** | `74e2f11a0543bd482c1d0539954588df1fb177d8f16ecc65bc14d573c44c219b` |
| **Handshake** | `Firmware: Marauder` — Probe still keys off those eight bytes |

Do not mix this image with a different FAP, or the v0.4 FAP with a different
image. On the Flipper, set **Settings → System → Log Device** to **Off**
(pins 13/14 are shared with the system log).

## Flash

This file is the **application partition**, not a merged factory image.

| Layout | App offset | Use |
|---|---|---|
| **SigRoam** (this image) | **`0x20000`** | App-slot update on a board already on the SigRoam partition table |
| **Marauder C5** | `0x10000` | Factory Scout Lite / Marauder installer — **not this file** |

Writing this bin at `0x10000`, or flashing it from the Marauder web page,
will not migrate a stock board. Converting a factory Marauder layout is not
a one-file flash in this repository.

**Unplug the Flipper** before connecting Scout Lite USB-C.

```bash
esptool --chip esp32c5 -p PORT -b 460800 write-flash 0x20000 sigroam_lite.bin
```

`PORT` is the Scout Lite USB-C port (`/dev/cu.usbmodem*` on macOS, `COMx` on
Windows, `/dev/ttyACM*` on Linux). After the write, unplug and reseat USB-C,
or hit reset, before mounting the board on the Flipper.

To go back to Marauder, use
[flash.pingequa.com/devices/scout-lite](https://flash.pingequa.com/devices/scout-lite)
(Marauder at `0x10000`). Do not then write this SigRoam app onto that layout.

## Two setups, never together

Scout Lite USB-C and Flipper 5 V OTG are physically exclusive. GPIO 13/14 are
muxed: USB data **or** GNSS UART, not both.

| | Flash / USB-C | Survey / Flipper |
|---|---|---|
| Scout USB-C | Connected | **Unplugged** |
| Flipper GPIO + 5 V | **Unplugged** | Mounted |
| GPIO 13/14 | USB D+/D− | GNSS |
| GPS | Off | On |

Plug USB-C while the Flipper is mounted, or mount the Flipper while USB-C is
plugged in, and you lose GNSS, the Flipper link, or both. That is expected.

Do not power the scanner from Flipper pin 9 (3.3 V).

## What it deliberately does not do

Receive-only is a product rule, not a missing feature:

- deauthentication / disassociation
- WPA handshake / PMKID capture
- evil twin, karma, rogue AP, SoftAP
- beacon spam / BLE spam
- password cracking
- `WIFI_MODE_AP` / `APSTA` / `esp_wifi_80211_tx`

If you need those, keep factory Marauder — or use GhostESP / Bruce. This
image will not grow them.

## Survey log

CSV is written on the **Scout Lite** microSD (FAT32), not on the Flipper card.
The FAP stores settings only. Upload the CSV at [wigle.net](https://wigle.net).

This image does **not** associate to a home AP and POST the file for you.
Pull the card.

## License

The files in this repository are MIT. See [LICENSE](LICENSE).
The scanner source is not published. The binary is provided as-is.

Finished Scout Lite boards are intentional radiators under FCC Part 15 / SDoC.
Educational and lawful network research only. Scan networks you own or are
explicitly authorized to assess.

## Links

- [SigRoam Wardriving (Flipper app)](https://github.com/pingequalab/sigroam-wardriving)
- [Scout Lite (board)](https://github.com/pingequalab/scout-lite)
- [PINGEQUA](https://pingequa.com)
- [ESP32 Marauder](https://github.com/justcallmekoko/ESP32Marauder)
- [WiGLE](https://wigle.net)

**PINGEQUA Lab** — hardware and firmware for RF, GPS, and field survey.
