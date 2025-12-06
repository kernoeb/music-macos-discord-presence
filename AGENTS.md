# Project: Music Discord Presence

macOS app that shows Telegram and YouTube Music playback as Discord Rich Presence.

## Tech Stack

- **Runtime**: Bun (not Node.js)
- **Language**: TypeScript
- **Discord**: @xhayper/discord-rpc
- **System Tray**: systray2
- **macOS APIs**: JXA (JavaScript for Automation) via osascript

## Key Commands

```sh
bun install              # Install dependencies
bun run tray             # Run tray app
bun run start            # Run CLI version
bun run build            # Compile standalone binary
bun run typecheck        # Type check
```

## Important Patterns

### systray2 Import (Bun compile fix)
```ts
// Must use require().default for Bun --compile compatibility
const SysTray = require("systray2").default;
```

### Environment
```sh
DISCORD_CLIENT_ID=xxx bun run tray
```

### JXA for Now Playing
Uses `osascript -l JavaScript` to query MediaRemote framework. This bypasses macOS 15.4+ entitlement restrictions.

### Media Source Detection
Supports multiple media sources:
- **Telegram**: Detected via bundle IDs (`ru.keepcoder.Telegram`, etc.)
- **YouTube Music PWA**: Detected via Chromium PWA bundle ID pattern (`*.app.cinhimbnkkaeohfgghhklpknlkffjgod`)

## File Structure

- `index.ts` - Main app (CLI + tray mode, supports --test flag)
- `start.sh` - Launcher script

## Gotchas

1. **systray2**: CommonJS module - use `require().default`, not ES import
2. **Discord RPC**: Connect in background with retry - don't block tray startup
3. **Artwork**: Fetched from Deezer first, then iTunes Search API, cached in memory
4. **Playback detection**: Check both `isPlaying` AND `playbackRate > 0`
5. **YouTube Music PWA**: Bundle ID includes browser prefix + extension ID suffix
