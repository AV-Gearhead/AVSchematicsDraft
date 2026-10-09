# Patchsheet – AV schematic drafting

A browser tool for drawing video/audio/control signal-flow schematics. Drag devices from a shared library onto an A3 sheet, drag from port to port to create cables, then download the drawing as PDF, PNG or SVG and the cable schedule as CSV.

No build step and no server code: the tool is two files, `tool/index.html` and `tool/devices.json`. The repo root `index.html` is an intro/landing page that links to it.

## Host it on GitHub Pages

1. Repo **Settings → Pages → Build and deployment → Deploy from a branch**, choose `main` and `/ (root)`, save.
2. The landing page goes live at the repo's Pages URL (or the custom domain in `CNAME`, currently `www.andytstool.com`), and the tool itself at `/tool/`.

Note: on GitHub Free/Team plans a Pages site is public. That's fine for the tool and a device library (it's all public datasheet info), and drawings never leave the user's browser unless they download them. If you want the site itself behind a login, GitHub Enterprise Cloud supports private Pages; Cloudflare Pages + Cloudflare Access or Netlify with password protection are alternatives.

## The shared device library ("cloud" library)

`tool/devices.json` in the repo **is** the cloud library. Everyone who opens the site loads the latest version, so updating the library = committing a change to that file.

Ways to add devices:
- **In the app:** click *New device* (or *Edit copy* on an existing one), fill in ports, save. It's stored in that browser. Click *Export library* to download a merged `devices.json`, then upload it to the repo as `tool/devices.json` (GitHub web UI → Edit / Upload). Use a pull request if you want an engineer to review port data first.
- **Directly in JSON:** edit `tool/devices.json` on GitHub.

To pull from more than one source (for example a company-wide repo plus a project-specific list), edit `CONFIG.libraryUrls` near the top of the script in `tool/index.html`. Each URL must return `{ "devices": [...] }` and allow cross-origin requests (raw.githubusercontent.com and Gist raw URLs do).

If you later want staff to add devices from inside the app without touching GitHub, swap the JSON file for a small database such as Supabase or Firebase: a `devices` table with a JSON `ports` column, read through its REST URL in `libraryUrls`, plus a login for write access.

### Device format

```json
{
  "id": "crestron-rmc4",
  "manufacturer": "Crestron",
  "model": "RMC4",
  "category": "Control",
  "tag": "CP",
  "description": "4-Series control system",
  "verified": true,
  "source": "https://link-to-datasheet",
  "ports": [
    { "id": "com", "name": "COM RS-232/422/485", "signal": "control", "dir": "io", "conn": "Euro 5-pin" },
    { "id": "ir1", "name": "IR 1", "signal": "control", "dir": "out", "conn": "Euro 4-pin" }
  ]
}
```

- `signal`: `video`, `audio`, `control`, `network`, `usb`, `power` (only matching signals can be cabled together)
- `dir`: `in`, `out` or `io` (bidirectional, for example LAN, RS-232, Q-SYS flex channels)
- `side` (optional): `left` or `right` to override the default (inputs left, outputs and bidirectional right)
- `verified`: set to `true` once someone has checked the ports against the datasheet. The app shows this as a badge.

## Using the tool

- Drag a device from the library onto the sheet (or double-click it). Drag its header to move it.
- Drag from a port's square terminal to another port to draw a cable. Cables get IDs automatically by signal type (V-001 video, A-001 audio, C-001 control, N-001 network…).
- Select a cable to set cable type, length and notes; select a device to change its tag, add a location note, or flip its sides.
- Drawing details (project, client, revision, drawn by) fill in the title block.
- **Download**: PDF uses the browser's print dialog (choose "Save as PDF", A3 landscape). PNG and SVG download directly; SVG opens in Illustrator, Inkscape, or Visio for touch-ups. The cable schedule exports as CSV for Excel.
- **File → Save project file** downloads the drawing as `.json` so colleagues can open and revise it. Work also autosaves in the browser.
