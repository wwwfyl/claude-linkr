# claude-linkr

A [Claude Code](https://claude.com/claude-code) skill for driving a remote screen through a [Radxa LinkR](https://github.com/radxa-linkr) KVM-over-IP device: screenshots, OCR, mouse clicks, typing and key presses, via a small `linkr` CLI.

## Install

```bash
git clone https://github.com/wwwfyl/claude-linkr ~/.claude/skills/linkr
ln -s ~/.claude/skills/linkr/scripts/linkr ~/.local/bin/linkr   # optional, puts linkr on PATH

mkdir -p ~/.config/linkr && chmod 700 ~/.config/linkr
printf %s 192.168.x.x > ~/.config/linkr/host
printf %s 'your_api_token' > ~/.config/linkr/token && chmod 600 ~/.config/linkr/token
```

Or set `LINKR_HOST` and `LINKR_TOKEN` in the environment. Set `LINKR_W`/`LINKR_H` if the target screen isn't 1920x1080.

Requirements: `curl`, `jq`; ImageMagick for cropped screenshots and OCR; `tesseract` for OCR.

## Usage

```bash
linkr shot /tmp/screen.jpg            # full screenshot
linkr shot /tmp/part.jpg 0 0 800 600  # cropped
linkr ocr
linkr click 960 540
linkr drag 960 800 960 300            # swipe up (one finger)
linkr type 'hello world'
linkr key ControlLeft KeyA
linkr scroll 5
```

Coordinates are pixels; key names are `KeyboardEvent.code` values. See [SKILL.md](SKILL.md) for what Claude is told.

Based on the API description in [radxa-linkr/linkr-skills](https://github.com/radxa-linkr/linkr-skills).

## License

MIT
