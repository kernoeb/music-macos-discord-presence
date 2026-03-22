# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

macOS app (15.4+) that displays Telegram and YouTube Music playback as Discord Rich Presence with album artwork. Single-file TypeScript app running on Bun.

## Commands

```sh
bun install              # Install dependencies
bun run start            # Run CLI version
bun run dev              # Run CLI with auto-reload (bun --watch)
bun run tray             # Run system tray app
bun run test             # Run in test mode (no Discord connection)
bun run build            # Compile standalone binary (~57MB)
bun run typecheck        # Type check (tsc --noEmit)
```

Environment: `DISCORD_CLIENT_ID=xxx` required (via `.env` file or inline).

## Architecture

Everything lives in `index.ts` (~830 lines), a single monolithic file with no modules.

**Data flow:** Media player → macOS MediaRemote (via JXA/osascript) → Artwork APIs → Discord RPC

**Key subsystems:**

1. **Media detection** — Executes JXA script via `osascript -l JavaScript` to query macOS MediaRemote framework, bypassing 15.4+ entitlement restrictions. Returns `NowPlayingInfo` with track metadata, playback state, and bundle identifier.

2. **Source identification** — Matches `bundleIdentifier` against known Telegram IDs or YouTube Music PWA pattern (`*.cinhimbnkkaeohfgghhklpknlkffjgod`). YouTube Music has a fallback that checks `app_mode_loader` PIDs via `ps` with a 30s TTL cache.

3. **Artwork fetching** — Multi-strategy with in-memory cache: Deezer API → iTunes Search API → YouTube thumbnail (for YT Music). Uses `normalizeSearchTerm()` for diacritics/punctuation and `matchesResult()` for fuzzy artist/title matching.

4. **Discord presence** — 5-second polling loop via `@xhayper/discord-rpc`. Calculates timestamps from system clock + elapsed time. Validates Discord field constraints (min 2 chars, max 128 chars).

5. **System tray** — `systray2` provides pause/resume/quit menu. Tray mode activates via `--tray` flag or `IS_COMPILED` compile-time define.

**Three run modes:** CLI (blocking Discord connect), Tray (async connect with retry), Test (`--test`, no Discord).

## Critical Patterns

- **systray2 must use CommonJS require**: `const SysTray = require("systray2").default` — ES import breaks Bun compilation.
- **Playback detection**: Must check both `isPlaying` AND `playbackRate > 0`.
- **Telegram voice message filtering**: `isTelegramVoiceMessage()` skips voice/video messages whose titles are date/time strings (e.g. "aujourd'hui à 20:12"). Pattern: short title (≤40 chars) ending with `HH:MM` (± AM/PM).
- **Build define**: `bun build --define IS_COMPILED=true` enables tray mode automatically in compiled binary.
- `start.sh` is the launcher: tries compiled binary first, falls back to `bun run index.ts --tray`.
