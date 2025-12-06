# 🎵 Music Discord Rich Presence

Display what you're listening to on Telegram or YouTube Music as Discord Rich Presence with album artwork.

![macOS](https://img.shields.io/badge/macOS-15.4+-blue)
![Bun](https://img.shields.io/badge/runtime-Bun-orange)

## Features

- 🎶 Detects music playing from Telegram and YouTube Music PWA on macOS
- 🎮 Shows track info as Discord Rich Presence (Listening activity)
- 🎨 Fetches album artwork automatically from iTunes/Deezer
- ⏱️ Displays progress bar with elapsed/remaining time
- 🔔 System tray app with pause/quit controls
- 📦 Standalone executable (no Bun required to run)
- ✅ Works on macOS 15.4+ (uses JXA to bypass MediaRemote restrictions)

## Supported Media Sources

| Source | Detection Method |
|--------|-----------------|
| **Telegram** | Native macOS app, Telegram Desktop |
| **YouTube Music** | PWA installed via Chrome, Brave, Edge, or other Chromium browsers |

## Prerequisites

- **macOS 15.4+** (uses JavaScript for Automation to access Now Playing info)
- **Discord** desktop app running
- **Telegram** for macOS and/or **YouTube Music PWA**

For development:
- **Bun** runtime

### Installing YouTube Music PWA

1. Open [music.youtube.com](https://music.youtube.com) in Chrome, Brave, or Edge
2. Click the install icon in the address bar (or Menu → Install YouTube Music...)
3. The PWA will be installed to `~/Applications/`

## Quick Start

### Using the standalone executable

1. Download or build the executable (see below)
2. Run it with your Discord Client ID:

```bash
DISCORD_CLIENT_ID=your_client_id ./music-discord-presence

```

A system tray icon will appear. Click it to pause/resume or quit.

### Using Bun (development)

```bash
# Install dependencies
bun install

# Run the tray app
DISCORD_CLIENT_ID=your_client_id bun run tray

# Or run the CLI version (no tray)
DISCORD_CLIENT_ID=your_client_id bun start
```

## Building the Standalone Executable

```bash
bun run build
```

This creates `music-discord-presence` - a single ~57MB executable that includes everything.

**Note:** The first run will extract the systray binary to `~/.cache/node-systray/`. Make sure this is writable.

## Discord Setup

### 1. Create a Discord Application

1. Go to [Discord Developer Portal](https://discord.com/developers/applications)
2. Click **"New Application"** and give it a name (e.g., "Music Presence")
3. Copy the **Application ID** (this is your Client ID)

### 2. Add Rich Presence Assets

In your Discord application:

1. Go to **Rich Presence** → **Art Assets**
2. Upload images with these names:
   - `telegram` - Badge icon for Telegram playback
   - `youtube-music` - Badge icon for YouTube Music playback

The large image will be the album artwork fetched automatically from iTunes/Deezer. If no artwork is found, it falls back to your source-specific asset.

### 3. Enable Activity Display

In Discord Settings → Activity Privacy:
- Enable "Display current activity as a status message"

## Usage

### Tray App (Recommended)

```bash
DISCORD_CLIENT_ID=your_client_id bun run tray
# or
DISCORD_CLIENT_ID=your_client_id ./music-discord-presence
```

The tray icon provides:
- **⏸️ Pause** - Temporarily stop updating Discord presence
- **▶️ Resume** - Resume presence updates
- **🚪 Quit** - Exit the app

### CLI Version

```bash
DISCORD_CLIENT_ID=your_client_id bun start
```

Press `Ctrl+C` to stop.

### Test Mode (No Discord)

```bash
bun run index.ts --test
```

This tests Now Playing detection without connecting to Discord.

## How It Works

```
┌─────────────────────┐     ┌────────────────────┐     ┌─────────────────┐     ┌───────────────┐
│  Telegram / YTMusic │────▶│  macOS MediaRemote │────▶│  iTunes/Deezer  │────▶│   Discord     │
│      (playing)      │     │    (via JXA)       │     │      API        │     │ Rich Presence │
└─────────────────────┘     └────────────────────┘     └─────────────────┘     └───────────────┘
```

1. **Media source** (Telegram or YouTube Music) plays audio and reports to macOS Media Remote
2. **JXA (JavaScript for Automation)** queries the MediaRemote framework
3. **iTunes/Deezer Search API** fetches album artwork based on track/artist
4. The app updates **Discord RPC** with track info, artwork, and progress

### macOS 15.4+ Compatibility

Starting with macOS 15.4, Apple introduced entitlement verification for the MediaRemote framework, breaking tools like `nowplaying-cli`. This app uses JavaScript for Automation (JXA) to access the `MRNowPlayingRequest` class directly, which bypasses these restrictions without requiring SIP to be disabled.

## Configuration

You can modify these constants in the source:

```typescript
// Polling interval (how often to check for changes)
const POLL_INTERVAL = 5000; // 5 seconds

// Telegram bundle identifiers (add more if needed)
const TELEGRAM_BUNDLE_IDS = [
  "ru.keepcoder.Telegram",  // Telegram for macOS (native)
  "org.telegram.desktop",   // Telegram Desktop
  "com.tdesktop.Telegram",  // Telegram Desktop (alternative)
];

// YouTube Music PWA extension ID (same across all Chromium browsers)
const YOUTUBE_MUSIC_EXTENSION_ID = "cinhimbnkkaeohfgghhklpknlkffjgod";
```

## Troubleshooting

### Discord not showing the status

1. Make sure Discord desktop app is running (not web version)
2. Check that your Client ID is correct
3. Go to Discord Settings → Activity Privacy → Enable "Display current activity as a status message"

### Shows game controller instead of music icon

The app sets `ActivityType.Listening` which should show a music icon. If you see a game controller, Discord may be caching an old state - wait a moment or restart Discord.

### Not detecting Telegram audio

1. Make sure Telegram is actually playing audio (not just paused)
2. Check if the Now Playing widget appears in Control Center
3. Try playing a music file (not voice message) shared in a Telegram chat

### Not detecting YouTube Music

1. Make sure YouTube Music is installed as a PWA (not just a browser tab)
2. Check that music is actively playing (not paused)
3. Run `bun run index.ts --test` to verify detection

### Elapsed time resets every few seconds

This happens if the media source doesn't properly report elapsed time. The app tries to use the system timestamp to calculate actual elapsed time.

### "EACCES: permission denied" for systray binary

Run this to fix permissions:
```bash
chmod +x ~/.cache/node-systray/*/tray_darwin_release
```

### "RPC_CONNECTION_TIMEOUT" error

- Make sure Discord desktop app is running
- Try restarting Discord
- Check if you're using Discord Canary/PTB (should work, but try stable if issues persist)

## Scripts

| Command | Description |
|---------|-------------|
| `bun start` | Run CLI version |
| `bun run tray` | Run tray app version |
| `bun run dev` | Run CLI with auto-reload |
| `bun run build` | Build standalone executable |

## Technical Details

This app uses:
- **Bun** as the JavaScript runtime and bundler
- **@xhayper/discord-rpc** for Discord Rich Presence
- **systray2** for macOS menu bar integration
- **osascript** with JXA to query macOS MediaRemote
- **iTunes Search API** and **Deezer API** for album artwork

## License

MIT
