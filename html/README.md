# BigBoi Web Interface

Redesigned web interface for the ESPuino-based BigBoi music player. This is a complete rewrite of `management.html` focused on simplicity for non-technical users.

## What Changed

The original ESPuino web interface was replaced with a clean, mobile-first UI. **No backend code was modified** -- the new frontend communicates with the same WebSocket and REST API endpoints.

### Files Modified

| File | Change |
|---|---|
| `management.html` | Complete rewrite (single-file app) |
| `locales/en.json` | Updated translation keys, renamed to "BigBoi" |
| `locales/de.json` | Same |
| `locales/fr.json` | Same |

### Files NOT Modified

- `../src/Web.cpp` -- backend untouched
- `../processHtml.py` -- build script untouched
- `../platformio.ini` -- build config untouched
- `accesspoint.html` -- AP setup page untouched
- `swagger.html`, `REST_API.yaml` -- API docs untouched

## Architecture

```
management.html (single file)
├── <style>    Pico CSS (from CDN) + custom styles
├── <svg>      Inline SVG icon sprite (no Font Awesome)
├── HTML       4 views: Player, Files, Cards, Settings
├── <dialog>   Modals: Link Card, File Picker, Confirm, Info
└── <script>   Vanilla JS (no jQuery, no Bootstrap, no jstree)

Build process (unchanged):
  html/management.html
    → processHtml.py minifies + gzips
    → HTMLbinary.h (C++ byte array)
    → Embedded in firmware flash
    → Served by AsyncWebServer
```

### Dependencies

| What | How loaded | Required? |
|---|---|---|
| Pico CSS (classless) | CDN | Yes (page styling) |
| i18next + HTTP backend | CDN | No (graceful fallback to English keys) |

All other dependencies from the original UI (jQuery, Bootstrap, jstree, Font Awesome, bootstrap-slider, jQuery UI) have been removed.

### Size

| | Original | Redesigned |
|---|---|---|
| Raw HTML | 132 KB | ~85 KB |
| Minified | 94 KB | ~70 KB |
| Gzipped (in firmware) | ~25 KB | ~17 KB |

## UI Structure

### Bottom Navigation (4 tabs)

**Player** -- the default landing page
- Cover art (tap to load)
- Track name, progress bar (tap to seek), play time
- Playback controls: first, prev, play/pause, next, last
- Volume slider
- Bedtime timer: collapsible row with quick presets (15m, 30m, 1h, 2h, this song, playlist)
- Quick-assign card: appears when an unknown card is placed

**Files** -- SD card browser
- Back button + breadcrumb path navigation
- Flat file list with compact action buttons (play, download, delete)
- Search filter
- File and directory upload

**Cards** -- RFID card management
- Card list showing ID, assigned file/folder, and play mode
- Edit and delete per card
- FAB button to link a new card
- Link Card dialog with file picker and smart play mode detection

**Settings** -- grouped into collapsible sections
- **Volume** -- startup volume, limits, remember volume, smooth quiet volume, pause when muted
- **Playback** -- pause on card lift, resume after power off, resume on card switch, auto-play, ignore re-tap, mono, turn-off timer
- **Sound** -- 3-band equalizer (bass/mid/treble), file ordering, subfolder depth
- **Lights** -- brightness, night light
- **Controls** -- BigBoi button mapping (left eye, right eye, nose, encoder) with short/long press and combos
- **Volume Knob** -- reverse direction
- **Battery** -- voltage thresholds and check interval (shown only if hardware reports it)
- **MQTT / FTP / Bluetooth** -- shown only if enabled in firmware
- **Action Cards** -- assign special actions (sleep, keylock, etc.) to cards
- **System** -- info, log, card backup/restore/erase, firmware update, restart, shutdown

## Key Design Decisions

**Consumer-friendly language** -- "RFID tags" became "Cards", "Audiobook mode" became "Story mode", "Deep Sleep" became "Bedtime" / "Turn off after idle", button names match physical placement (Left Eye, Nose, Right Eye).

**Smart play mode defaults** -- when assigning a card, the play mode auto-selects based on what you picked: single file = "Play once", folder = "Story mode", .m3u = "Playlist file", http URL = "Web radio". Each mode shows a description box explaining what it does.

**Unknown card detection** -- when an unrecognized card is placed on the reader, a toast appears and the Link Card dialog auto-opens with the card number pre-filled.

**Dark mode default** -- dark theme by default, with `prefers-color-scheme` support and localStorage override.

**WebSocket resilience** -- all sends go through a `wsSend()` wrapper that checks `readyState` before sending, preventing the `DOMException` crashes that occurred with the volume slider on unstable connections. Success toasts only appear for explicit save actions, not volume/playback commands.

**No backend changes** -- uses the exact same WebSocket protocol and REST endpoints. The `processHtml.py` build script works identically.

## Development

To test locally without flashing:

1. Set `var remoteHost = "192.168.x.x"` at the top of `management.html` to your BigBoi's IP
2. Open `management.html` directly in a browser
3. Disable CORS in your browser (e.g., with a browser extension)

The build process is automatic via PlatformIO:
```
pio run --environment complete
```
`processHtml.py` runs as a pre-build step, minifying and gzipping all HTML files into `HTMLbinary.h`.

## Branch Info

- Branch: `ui-redesign`
- Based on: `master`
- Upstream: `biologist79/ESPuino`
- Only `html/` directory is modified, keeping merge conflicts minimal
