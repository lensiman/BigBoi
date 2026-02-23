# BigBoi - Card-controlled Music Player for Kids

A compact, friendly music player controlled by placing cards on top. Built on the [ESPuino](https://github.com/biologist79/ESPuino) platform with a redesigned web interface focused on simplicity.

![BigBoi](https://forum.espuino.de/uploads/default/original/2X/c/c6e89f4c0f06b4483a70b9d4f63918c77dd25e2b.jpeg)

## What is BigBoi?

BigBoi is a 9.5cm x 10cm x 9cm music player designed for children. Place a card on top and music starts playing. Swap the card, and a different album or story begins. No screens, no complicated menus -- just cards and music.

- **3 buttons**: Left eye (previous), nose (play/pause), right eye (next) -- all backlit
- **Volume knob**: Rotary encoder on the side
- **PN5180 RFID reader**: Reads cards through the enclosure
- **Speaker**: Visaton BF37 4 ohm with 3D-printed grille
- **Battery powered**: LiFePO4/LiPo with USB-C charging
- **Web interface**: Manage cards, upload music, and configure settings from your phone

More photos and build details: [BigBoi on the ESPuino Forum](https://forum.espuino.de/t/zeigt-her-eure-espuinos/554/251)

## Web Interface

BigBoi features a redesigned web interface with 4 tabs:

- **Player** -- playback controls, volume, bedtime timer, card detection
- **Files** -- browse and upload music to the SD card
- **Cards** -- manage which card plays which music
- **Settings** -- volume limits, playback behavior, lights, WiFi, and more

The interface is available in English, German, and French. It's designed for non-technical users: "RFID tags" are called "cards", "audiobook mode" is "story mode", and "deep sleep" is "bedtime".

See [`html/README.md`](html/README.md) for technical details about the web interface.

## Hardware

BigBoi uses the [ESPuino Complete](https://forum.espuino.de/t/espuino-complete/3817) PCB with:

- ESP32-WROVER (PSRAM required)
- PN5180 RFID reader (configured in `src/settings.h`)
- MAX98357a amplifier (integrated on PCB)
- SD card via SDMMC (1-bit mode)
- 3 NeoPixels (eyes and nose, backlit)
- PCA9555 port expander (integrated on PCB)
- LiFePO4 battery with voltage monitoring

> The hardware design has been built multiple times for friends. The enclosure is 3D-printed.

## Getting Started

### Prerequisites

- [Visual Studio Code](https://code.visualstudio.com/) with [PlatformIO](https://platformio.org/install/ide?install=vscode)
- [Git](https://git-scm.com/downloads)

### Clone and Build

```bash
git clone https://github.com/lensiman/BigBoi.git
cd BigBoi
```

### Configure

1. Edit `src/settings.h`:
   - Set `HAL` to `6` (ESPuino Complete) -- this is the default for the `complete` environment
   - Uncomment `#define RFID_READER_TYPE_PN5180` and comment out `RFID_READER_TYPE_MFRC522_SPI`
   - Enable/disable features: `BLUETOOTH_ENABLE`, `MQTT_ENABLE`, `FTP_ENABLE`, etc.

2. The board-specific pin config is in `src/settings-complete.h` (loaded automatically for HAL 6).

### Flash

Connect BigBoi via USB-C and run:

```bash
pio run --environment complete --target upload --target monitor
```

### First Boot

1. BigBoi creates a WiFi access point -- connect to it and enter your home WiFi credentials
2. After reboot, open BigBoi's IP address in a browser (shown in serial console)
3. Upload music via the **Files** tab
4. Place a card and assign music via the **Cards** tab

## How It Works

### Cards

Place a card on BigBoi to start music. The web interface lets you link any card to:

| Mode | What it does |
|---|---|
| **Story mode** | Plays folder in order, remembers where you left off |
| **Play folder** | Plays all tracks in order |
| **Shuffle folder** | Plays all tracks randomly |
| **Play once** | Plays a single track |
| **Repeat song** | Loops a single track |
| **Lullaby** | Picks one random song, then turns off |
| **Web radio** | Plays a web stream |
| **Playlist** | Plays items from a .m3u file |

Additional variants with subfolders, looping, and random albums are available in the web interface.

### Action Cards

Special cards that modify behavior instead of playing music:

| Action | Effect |
|---|---|
| Bedtime (15m/30m/1h/2h) | Auto-off after the set time |
| End of track / playlist | Auto-off when current track or playlist finishes |
| Keylock | Lock/unlock all buttons |
| Night mode | Dim the lights |
| Loop track / playlist | Toggle looping |

### Buttons (BigBoi layout)

| Button | Short press | Long press |
|---|---|---|
| **Left Eye** | Previous track | First track |
| **Right Eye** | Next track | Last track |
| **Nose** | Play / Pause | Play / Pause |
| **Volume Knob** (turn) | Volume up/down | -- |
| **Volume Knob** (press short) | Battery status | -- |
| **Volume Knob** (press long) | Power off | -- |

Button actions are fully customizable via the web interface under Settings > Controls.

### Lights

BigBoi has 3 NeoPixels (eyes and nose). They indicate:

- **Idle**: slow rotation (white = WiFi connected, green = WiFi off)
- **Playing**: track progress
- **Volume change**: green-to-red gradient
- **Paused**: orange
- **Low battery**: red flashes
- **Error**: red flash

Brightness and night mode are adjustable under Settings > Lights.

### Bedtime

Set a bedtime timer directly from the player view -- tap the bedtime bar and pick a duration (15m, 30m, 1h, 2h) or trigger (end of this song, end of playlist). BigBoi dims the lights and powers off automatically.

There's also an auto-off timer under Settings > Playback that turns BigBoi off after a period of inactivity.

## Uploading Music

- **Web interface**: Use the Files tab to browse the SD card and upload files/folders
- **FTP**: Enable FTP under Settings, then connect with [FileZilla](https://filezilla-project.org/) (default: `esp32`/`esp32`)
- **Physically**: Remove the SD card and copy files from your computer

Supported formats: MP3, OGG, WAV, FLAC, AAC, OPUS, M4A

## Advanced Features

### WiFi

WiFi is required for the web interface, FTP, MQTT, and web radio. It can be toggled on/off via an action card or button combo.

### Bluetooth

> Bluetooth and WiFi cannot run simultaneously due to memory constraints.

- **BT Speaker mode**: Stream from your phone to BigBoi
- **BT Headphone mode**: Stream from BigBoi to Bluetooth headphones

Switch modes under Settings > Bluetooth.

### MQTT / Smart Home

BigBoi can be controlled via MQTT for integration with Home Assistant, openHAB, or Node-RED. Enable under Settings > MQTT. See the [MQTT topic reference](https://github.com/biologist79/ESPuino#mqtt-topics) in the upstream docs.

### REST API

A full [REST API](./REST-API.yaml) is available for programmatic control.

### Firmware Updates

Upload new firmware directly from the web interface under Settings > System > Firmware Update (requires 16MB flash with OTA support).

## Credits

BigBoi is a fork of [ESPuino](https://github.com/biologist79/ESPuino) by [biologist79](https://github.com/biologist79) and the ESPuino community. The underlying firmware, hardware designs, and [community forum](https://forum.espuino.de) are their work.

This fork adds:
- Redesigned web interface optimized for non-technical users
- Consumer-friendly naming (Cards, Story mode, Bedtime, etc.)
- BigBoi-specific button and LED configuration
- German and French translations

Licensed under [GPL-3.0](LICENSE).
