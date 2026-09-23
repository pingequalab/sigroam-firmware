<p align="center">
  <img src="https://raw.githubusercontent.com/pingequalab/sigroam-wardriving/main/screenshots/dashboard.png" width="420" alt="SigRoam dashboard on Flipper Zero">
</p>

<h1 align="center">SigRoam</h1>

<p align="center">
  <b>Receive-only wardriving firmware for Flipper Zero.</b><br>
  Developed against <a href="https://www.pingequa.com/products/scout-lite">Scout Lite</a>
  · by <a href="https://www.pingequa.com">PINGEQUA Lab</a>
</p>

<p align="center">
  <a href="https://www.pingequa.com/products/scout-lite"><img src="https://img.shields.io/badge/buy-Scout%20Lite-orange" alt="Buy Scout Lite"></a>
  <a href="https://www.pingequa.com"><img src="https://img.shields.io/badge/site-pingequa.com-lightgrey" alt="pingequa.com"></a>
  <img src="https://img.shields.io/badge/host-Flipper%20Zero-blue" alt="Host: Flipper Zero">
  <img src="https://img.shields.io/badge/radio-receive--only-brightgreen" alt="Receive-only">
  <a href="https://go.pingequa.com/sr1g"><img src="https://img.shields.io/badge/link-go.pingequa.com%2Fsr1g-lightgrey" alt="go.pingequa.com/sr1g"></a>
</p>

<p align="center">
  <a href="https://www.pingequa.com/products/scout-lite"><b>Get Scout Lite</b></a>
  ·
  <a href="https://www.pingequa.com/blogs/guides-tutorials/wardriving-explained-flipper-zero"><b>How Flipper wardriving works</b></a>
  ·
  <a href="https://go.pingequa.com/sr1g"><b>go.pingequa.com/sr1g</b></a>
</p>

---

The Flipper Zero has **no Wi-Fi radio**. Wardriving on this platform is always
a split: a GPIO module does the scan, GNSS, and log; the Flipper is the
control head.

**SigRoam** is PINGEQUA Lab’s stack for that split — a dedicated scanner on
the module, a dedicated Flipper app, one job. This repository is the
**scanner firmware**. It is developed for Flipper Zero and built on
**[Scout Lite](https://www.pingequa.com/products/scout-lite)**.

This is **not** product v1.0. A download is not a claim that the device is
field-ready.

## Purpose

A Flipper wardrive should be a **survey**: dual-band Wi-Fi, BLE observation,
a GNSS tag, a WiGLE CSV on the module. It should not be an attack menu with
a wardrive command in it.

| | |
|---|---|
| **Host** | Flipper Zero — start / stop, live Dash / GPS / Sess, raw UART |
| **Scanner** | This firmware — 2.4 / 5 GHz passive scan, BLE, GNSS gate, microSD |
| **Board** | [Scout Lite](https://www.pingequa.com/products/scout-lite) — C5 + L86 + microSD |
| **Never** | Deauth, handshake, evil twin, SoftAP, beacon spam, `esp_wifi_80211_tx` |

Receive-only is a product rule. It will not grow attack features.

## Flipper wardriving compared

Flipper cannot wardrive Wi-Fi by itself. Native GPS is location only.
[Subdriving](https://www.pingequa.com/blogs/guides-tutorials/what-is-subdriving-flipper-zero)
is Sub-GHz, not Wi-Fi. Every Wi-Fi wardrive on this platform is an **app +
module** pair.

### Flipper apps

| | **SigRoam** | **[ESP32] WiFi Marauder** | **Ghost ESP** |
|---|---|---|---|
| Job | Survey dashboard | Full Marauder console | Pentest console |
| Wardrive | The whole app | One menu (`Wardrive`) | One feature among many |
| Attack controls | None | Deauth, portal, capture, … | Deauth, portal, spam, … |
| Live survey | Dash / Strm / GPS / Sess | Console / sniffer screens | Ghost menus |
| Official FW | Yes — FAP v0.5 | Lab / CFW bundle | Companion FAP |
| Scanner it drives | This image, or factory Marauder | Marauder on the module | GhostESP on the module |

### Flipper modules (Wi-Fi wardrive)

| | **Scout Lite + SigRoam** | **Wi-Fi Devboard** | **C5 multi-radio** (Apex 5, Rabbit-Labs, …) |
|---|---|---|---|
| Chip | ESP32-C5 2.4 + 5 GHz | ESP32-S2 **2.4 only** | ESP32-C5 2.4 + 5 GHz |
| GNSS | Onboard L86-M33 | Optional add-on | Usually onboard |
| microSD | Onboard | Optional breakout | Usually onboard |
| Extra radios | None | None | Sub-GHz / nRF24 typical |
| Module firmware | **SigRoam** (this) or factory toolkit | Marauder `_flipper.bin` | Marauder / GhostESP |
| Flipper UI | SigRoam FAP | Marauder companion | Marauder or Ghost companion |
| Built for | Wardrive only | Toolkit | Toolkit + extra radios |

Scout Lite is the only board this image supports. Other C5 modules are not
a drop-in.

Sources, 2026-09-18:
[Marauder × Flipper wiki](https://github.com/justcallmekoko/ESP32Marauder/wiki/Flipper-Zero) (2026-09-15);
[Flipper Lab · WiFi Marauder](https://lab.flipper.net/apps/esp32_wifi_marauder);
[GhostESP Flipper companion](https://github.com/GhostESP-Revival/GhostESP-FlipperCompanion);
[Wi-Fi Devboard flash notes](https://flash.pingequa.com/devices/flipper-wifi-devboard-marauder) (2026-09-05);
[C5 module roundup](https://www.pingequa.com/blogs/guides-tutorials/scout-lite-vs-apex-5-flipper-wardriving) (updated 2026-08-03).

## Stack (Flipper + Scout Lite)

```
  Flipper Zero                      Scout Lite
  ┌──────────────────────┐          ┌──────────────────────────┐
  │ SigRoam FAP v0.5     │  UART    │ SigRoam firmware (this)  │
  │ Dash · GPS · Sess    │◄────────►│ C5 · L86 · microSD       │
  └──────────────────────┘  13/14   └──────────────────────────┘
```

| | |
|---|---|
| **Board** | [Scout Lite](https://www.pingequa.com/products/scout-lite) |
| **SoC** | ESP32-C5-WROOM-1U-N8R8 |
| **GNSS** | Quectel L86-M33 (GPS / GLONASS / Galileo) |
| **Log** | Module microSD · WigleWifi CSV + session manifest |
| **Pins** | Flipper 13/14 UART · 5 V on pin 1 |
| **Handshake** | `Firmware: Marauder` — FAP v0.5 pairs without a protocol break |

Factory Scout Lite still ships a general toolkit image. This flash is
**optional**. Do not mix this image with a different FAP.

On the Flipper: **Settings → System → Log Device → Off**.

## Now

| | |
|---|---|
| **Version** | **0.5** (pairs with FAP v0.5; **not** product v1.0) |
| **Release** | [SigRoam 0.5 for Scout Lite](https://github.com/pingequalab/sigroam-firmware/releases/tag/v0.5) |
| **Tag** | `v0.5` (formal) · `app-13e13033` (app SHA prefix) |
| **File** | `sigroam_lite.bin` · 1638160 B |
| **SHA-256** | `13e13033ad2f124b6edb4c599b99e10fb576872964169ea627fb8b320ea54b7c` |
| **Write** | Bootloader, partition table, and otadata are the v0.4 bytes. The new file is the app at **`0x20000`**. |
| **FAP** | [sigroam-0.5.fap](https://github.com/pingequalab/sigroam-wardriving/releases/tag/v0.5) |
| **Flasher** | [flash.pingequa.com/devices/scout-lite](https://flash.pingequa.com/devices/scout-lite) |

This image observes BLE and uploads a sealed WiGLE CSV from the scanner. HTTP 429 is WiGLE's daily file limit. The round stops and is not marked done.

UART handshake is still `Firmware: Marauder` / `Version: v1.14.1-sigroam-0`. Not a finished product.

```text
shasum -a 256 sigroam_lite.bin
# 13e13033ad2f124b6edb4c599b99e10fb576872964169ea627fb8b320ea54b7c
```

Previous app image, v0.4: tag `v0.4` / `app-d4a21243`, 1399424 B, SHA-256 `d4a212438d5b34c5853646de22506ff8a939e36c960b538d26bd4b6d26663fae`. Older app image: tag `app-249eda5e`, 1398400 B, SHA-256 `249eda5e681368a57fb8d998e9926fe10875af2077bcbc6a9e30a68554adf502`.

## Upload

A sealed survey file can go to WiGLE from the scanner. That step does not
use a computer.

The file is the WigleWifi CSV from the survey: 2.4 GHz, 5 GHz, and BLE
rows. A successful upload keeps WiGLE's transaction id with that file.

HTTP 429 is WiGLE's daily file limit. The round stops and is not marked
done. On the Flipper, Upload shows `WiGLE busy`. The card can still be
copied and sent by hand at [wigle.net](https://wigle.net). A hand upload
from a computer is a different path.

## Next (Flipper)

Scout Lite stays the reference Flipper module. The Flipper stays the host.
Receive-only stays.

Public direction — not a ship date, not v1.0:

1. **Keep Flipper first.** This firmware is a Flipper scanner, not a
   standalone gadget OS.
2. **Keep Scout Lite first.** New Flipper modules, if any, speak this
   scanner. They do not fork a second capture stack.
3. **Keep the survey UI on the Flipper.** FAP remains the field dashboard
   (Official firmware included).
4. **Survey loop.** A sealed CSV can upload from the scanner. HTTP 429
   means WiGLE's daily file limit. You can still copy the card and upload
   at [WiGLE](https://wigle.net).
5. **Install path.** Browser flasher: [flash.pingequa.com/devices/scout-lite](https://flash.pingequa.com/devices/scout-lite). SigRoam 0.5 is first in the picker.

Unlisted boards are unsupported.

## Flash

Application partition only (`ota_0` at `0x20000`). Not a merged factory
image. **Unplug the Flipper** before Scout Lite USB-C.

```bash
esptool --chip esp32c5 -p PORT -b 460800 write-flash 0x20000 sigroam_lite.bin
```

USB-C flash and Flipper 5 V survey are exclusive. GPIO 13/14 is USB **or**
GNSS, not both. Do not power the scanner from Flipper pin 9.

Factory recover:
[flash.pingequa.com/devices/scout-lite](https://flash.pingequa.com/devices/scout-lite)
(toolkit image at `0x10000` — will not install SigRoam).

## License

Repository files: [MIT](LICENSE). Scanner source is not published.
Binary as-is.

Finished Scout Lite boards are intentional radiators under FCC Part 15 / SDoC.
Educational and lawful network research only.

---

<p align="center">
  <a href="https://www.pingequa.com/products/scout-lite"><b>Get Scout Lite — pingequa.com</b></a><br>
  <a href="https://www.pingequa.com/blogs/guides-tutorials/how-to-first-wardrive-flipper-zero-scout-lite">First wardrive</a>
  ·
  <a href="https://www.pingequa.com/blogs/guides-tutorials/flipper-zero-wardriving-app-sigroam">SigRoam on Flipper</a>
  ·
  <a href="https://go.pingequa.com/sr1g">go.pingequa.com/sr1g</a>
</p>

<p align="center"><b>PINGEQUA Lab</b> — Flipper survey hardware and firmware.</p>
