---
name: screenshot
description: Use when the user explicitly asks for a desktop or system screenshot (full screen, specific app or window, or a pixel region), or when tool-specific capture capabilities are unavailable and an OS-level capture is needed.
---

# Screenshot Capture

Use OS-level capture when the user asks for a screenshot or when tool-specific capture is unavailable.

## Save Location Rules
1. If the user specifies a path, save there.
2. If the user asks for a screenshot without a path, use the OS default screenshot location.
3. If Codex needs a screenshot for its own inspection, save to a temp directory.

## Tool Priority
- Prefer tool-specific capture tools (Figma MCP, Playwright, app-specific exporters).
- Use this skill for full desktop, system-level, or app/window captures.

## macOS Preflight
Run once before window/app capture:
```bash
bash <path-to-skill>/scripts/ensure_macos_permissions.sh
```

## Preferred Helper (macOS/Linux)
Run from repo root:
```bash
python3 <path-to-skill>/scripts/take_screenshot.py
```

Common options:
- Temp output: `--mode temp`
- Specific path: `--path output/screen.png`
- App capture (macOS): `--app "AppName"`
- Window title (macOS): `--window-name "Settings"`
- Active window: `--active-window`
- Region: `--region 100,200,800,600`
- List windows (macOS): `--list-windows --app "AppName"`

## Windows Helper
```powershell
powershell -ExecutionPolicy Bypass -File <path-to-skill>/scripts/take_screenshot.ps1
```

Common options:
- Temp output: `-Mode temp`
- Specific path: `-Path "C:\Temp\screen.png"`
- Active window: `-ActiveWindow`
- Region: `-Region 100,200,800,600`
- Window handle: `-WindowHandle 123456`

## Fallback OS Commands
Use only if helpers are unavailable.

macOS:
```bash
screencapture -x output/screen.png
screencapture -x -R100,200,800,600 output/region.png
screencapture -x -l12345 output/window.png
```

Linux (first available):
```bash
scrot output/screen.png
```
```bash
gnome-screenshot -f output/screen.png
```
```bash
import -window root output/screen.png
```

## Linux Tool Selection
The helper picks the first available tool: `scrot`, `gnome-screenshot`, then ImageMagick `import`.
If none exist, ask the user to install one and retry.

## Multi-Display Notes
- macOS: one file per display for full-screen capture.
- Linux/Windows: full-screen uses virtual desktop; use `--region` to isolate.

## Previewing and Comparison
- If the screenshot is for validation, open it with the image viewer tool.
- When comparing design vs implementation, capture both and compare raw images first.
- Avoid editing or annotating unless asked.

## Region Selection Tips
- Use even pixel dimensions when possible.
- Capture a small margin around the target element to show context.
- If the user is unsure, suggest capturing a larger region and cropping later.

## Error Handling
- If macOS app/window capture returns no matches, run `--list-windows` and retry.
- If Linux capture fails, check `command -v scrot`, `gnome-screenshot`, `import`.
- If capture fails due to permissions, ask the user to grant screen recording or retry.

## Output Expectations
- Always report the saved file path(s).
- If multiple images are produced, list them in order.

## Definition of Done
- Screenshot captured at the requested scope and saved correctly.
- Path(s) reported to the user.
