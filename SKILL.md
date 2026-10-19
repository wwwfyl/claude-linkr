---
name: linkr
description: Look at and drive a remote machine's screen through a Radxa LinkR KVM-over-IP device - take screenshots, OCR the screen, click, type, press keys and scroll. Use when testing or operating a GUI on a machine whose HDMI and USB HID are connected to a LinkR, or when the user mentions LinkR, KVM, "click on the robot screen", or a screenshot of the remote display.
allowed-tools: Bash(linkr:*), Bash(*/scripts/linkr:*)
---

# LinkR KVM

`scripts/linkr` (next to this file) wraps the LinkR public HTTP API. If `linkr` is on `PATH`, call it directly; otherwise use the full path to `scripts/linkr`.

## Setup

Needs `curl`, `jq`; `shot` with a crop region needs ImageMagick (`convert`), `ocr` needs ImageMagick and `tesseract`.

Configuration, environment first, then files:

- `LINKR_HOST` or `~/.config/linkr/host` — device IP/hostname
- `LINKR_TOKEN` or `~/.config/linkr/token` — API token from the LinkR web UI (keep the file `chmod 600`)
- `LINKR_W` / `LINKR_H` — target screen size in pixels, default `1920`x`1080`

If neither is set, ask the user for the address and token, then write them to the config files. Never echo the token back or commit it.

## Commands

| Command | What it does |
|:---|:---|
| `linkr shot [FILE [X1 Y1 X2 Y2]]` | Save a JPEG screenshot (default `/tmp/linkr.jpg`), optionally cropped; prints the path. Read the file to see it. |
| `linkr ocr` | Screenshot + local tesseract; prints recognised lines. |
| `linkr move X Y` | Move the pointer. |
| `linkr click X Y [left\|right\|middle]` | Move and click. |
| `linkr type TEXT` | Type ASCII text (printable 32–126, Tab, newline). |
| `linkr key KEY...` | Press keys together, release in reverse: `linkr key Enter`, `linkr key ControlLeft KeyA`. |
| `linkr scroll DY` | Scroll the wheel, `-20..20`, positive is down. |

Coordinates are screen pixels (origin top-left); the script converts them to the 0..1 fractions LinkR expects, so they must match `LINKR_W`/`LINKR_H`. Key names are [`KeyboardEvent.code`](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/code/code_values) values.

## Working loop

1. `linkr shot` and read the image before acting — find targets from the actual screen, not from memory.
2. Act (`click`, `type`, `key`), then `sleep` long enough for the UI to react.
3. Screenshot again (crop to the area of interest to save tokens) and confirm the result before the next step.

Chain steps with `&&` in one Bash call: `linkr click 960 453 && sleep 2 && linkr shot /tmp/s.jpg`.

## Gotchas

- `type` is slow: LinkR drops characters unless text is sent in ~30-character chunks with a 1 s pause after each, which the script does for you. Non-ASCII text cannot be typed at all.
- The first HID event after a long idle can be lost — verify with a screenshot.
- `ocr` thresholds the image for light text on dark UIs; on light themes it may read poorly — read the screenshot instead.
- Don't race on-screen countdowns (auto-reboot timers, etc.) with scripted clicks; ask the user or disable the auto action first.
- Clicks have real effects on a real machine. Ask before destructive actions (delete, reset, reboot, shutdown).

## Raw API

For anything the script doesn't cover:

- `GET http://$HOST/api/public/snapshot` → JPEG
- `POST http://$HOST/api/public/control` with `{"events": [...]}` → `{"code": 0, ...}` on success
- Header `Authorization: token $TOKEN` on both

Events: `["keyboard", code, pressed]`, `["mouse_abs", buttons, x, y, wheelY, wheelX]` (x, y in 0..1; buttons bitmask 1 left, 2 right, 4 middle), `["mouse_rel", buttons, dx, dy, wheelY, wheelX]`, `["text", "..."]` (≤1024 chars), `["delay", ms]`.
