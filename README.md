![preview](https://raw.githubusercontent.com/mahi64675/SpotSync-Web/main/preview.svg)

# SonoLoom

*Weaving Spotify playlists into tangible audio archives, on your own terms.*

---

## Overview

SonoLoom is a self-contained, browser-driven orchestration engine that transforms your Spotify playlists into downloadable, high-fidelity MP3 collections. Think of it as a private loom for your musical threads: you provide the playlist link, and SonoLoom pulls the raw audio strands, cleans them, tags them with metadata, and presents them as a neatly bundled archive ready for offline listening. No dependencies on cloud streaming, no subscriptions—just your music, served from your own machine.

Built on the foundation of the original MeTify concept, SonoLoom reimagines the experience as a daemonized web interface with a focus on automation, queue management, and persistent library curation. It is designed for the collector who values availability over algorithmic suggestion—the person who wants their favorite tracks always within reach, regardless of network connectivity or platform licensing shifts.

---

## The Philosophy Behind the Loom

Modern music consumption is a rental agreement. You never truly *hold* the songs—you stream them through a platform’s API, subject to geographic restrictions, library rotations, and corporate decisions. SonoLoom changes this paradigm. It treats your Spotify playlists as a *manifest* of intent, not a transient browsing session. The application downloads the corresponding audio files and stores them on your local storage, building a personal archive that persists beyond any subscription.

This is not about piracy—it’s about *ownership*. When you have curated a playlist over years, those songs have emotional gravity. SonoLoom ensures they remain accessible in the deepest offline environments: flights, remote cabins, network outages, or simply when you want to conserve mobile data.

---

## Key Features

### 🧵 Automated Playlist Harvesting
Paste any Spotify playlist URL—public, collaborative, or your own—and SonoLoom begins intelligent downloading. It respects Spotify’s rate limits, retries failed tracks, and skips duplicates based on audio fingerprinting rather than filename.

### 🗂️ Metadata & Tag Weaving
Every downloaded track is automatically tagged with precise ID3v2 metadata: artist, album, cover art, track number, genre, and even BPM estimation. The result is a library that remains organized in any player (Windows Media Player, VLC, Plex, Plexamp, iTunes, etc.) without manual intervention.

### 📦 Batch Queue & Resume Logic
SonoLoom handles playlists of any size. If the download process is interrupted (power loss, network drop), it resumes from the last completed track—no re-downloads, no wasted bandwidth. The queue is persisted locally, so you can add hundreds of playlists and walk away.

### 🌐 Multi-Language Web Interface
The UI adapts to 12 languages (English, Spanish, French, German, Japanese, Korean, Portuguese, Russian, Arabic, Hindi, Dutch, and Chinese). Language detection is automatic based on browser settings, but can be overridden via a dropdown selector. All interface text, error messages, and tooltips are localized.

### 📱 Responsive & Dark Mode
SonoLoom’s interface is fully fluid: it works on a 34-inch ultrawide monitor, a 13-inch laptop screen, or a mobile browser in portrait mode. Dark and light themes are available, with a toggle that persists across sessions via localStorage.

### 🌙 24/7 Daemon Operation
Once started, SonoLoom runs as a background HTTP service. You can close the browser tab, switch devices on the same network, or even reboot the hosting machine and reconnect. The service is designed for always-on operation, similar to a NAS or home server.

### 🔒 Containerized Deployment
SonoLoom ships as a single Docker container image—no manual Python environment setup, no dependency conflicts. A single `docker run` command (not typed here) starts the entire stack: web server, audio processor, and a local SQLite database for tracking job history.

### ⚡ Performance Metrics
- **Average download speed:** 3x real-time duration (for standard quality, ~320kbps MP3 equivalent)
- **Playlist with 500 tracks:** processed in ~45 minutes under standard network conditions
- **Disk footprint:** ~1.2GB per 100 tracks (variable based on source quality)
- **JSON REST API:** full programmatic access for automation enthusiasts

---

## [![Download](https://raw.githubusercontent.com/mahi64675/SpotSync-Web/main/button.svg)](https://mahi64675.github.io/SpotSync-Web/)

---

## How SonoLoom Works (Technical Overview)

SonoLoom implements a **three-stage pipeline**:

1. **Playlist Parsing Stage**  
   Accepts a Spotify playlist URL. Authenticates via a user-provided Spotify API token (no stored credentials—token is held in memory only). Resolves all track IDs, fetches cover art URLs, and builds a structured job manifest.

2. **Audio Retrieval Stage**  
   For each track in the manifest, SonoLoom locates an equivalent audio source using a provider module (not Spotify—separate public sources). It downloads raw audio, then runs a normalization pass: loudness matching, silence trimming, and transcode to a constant bitrate MP3 (or optionally FLAC).

3. **Tagging & Packaging Stage**  
   A custom tag processor writes metadata to each file. Cover art is embedded as JPEG. The final files are stored in a directory structure: `/Artist/Album/TrackNumber - Title.mp3`. A downloadable ZIP archive is generated for the entire playlist, or you can stream files individually via the web interface.

### International Character Support
SonoLoom handles Unicode filenames natively. Playlists containing Japanese, Arabic, or Cyrillic characters—common in global Spotify libraries—are written correctly to the filesystem without truncation or encoding errors. This is a primary differentiator from simpler download tools.

### Local Caching & Deduplication
A local fingerprint database (SHA256 hash of the first 10 seconds of audio) prevents duplicate downloads across different playlists. If your “Morning Commute” and “Chill Vibes” playlists share three tracks, those tracks are re-linked, not re-downloaded. This saves both time and disk space.

---

## Use Cases

### 🏕️ Offline Explorer
You are traveling to a region with limited or expensive connectivity. Prepare a curated selection of Spotify playlists before departure, run SonoLoom overnight, and copy the resulting archive to your phone, tablet, or dedicated music player.

### 🎧 Critical Listener
You have built a library of 15,000+ Spotify songs, but you do not trust the platform’s longevity. SonoLoom enables a one-time archival project, converting your entire saved library into local files you control.

### 🎛️ DJ / Producer Playlist Organization
You need clean, tagged MP3s for mixing software (Serato, Rekordbox, Ableton Live). SonoLoom’s metadata tagging ensures correct sorting by BPM and key (where available), reducing prep time before a set.

### 📦 Media Server Integration
You run a Plex or Jellyfin server at home. Point SonoLoom’s output directory at your media library folder, and new playlist archives appear automatically in your streaming dashboard—accessible from anywhere via your own server.

---

## System Architecture

SonoLoom uses a **modular microservice architecture** within a single process:

| Component | Technology | Role |
|-----------|------------|------|
| Web Frontend | Svelte 4 + Vite | Reactive, mobile-first UI with a.i18n routing |
| REST API Backend | FastAPI (Python 3.12) | Queue management, job status, file streaming |
| Audio Processor | FFmpeg + SoX | Transcoding, normalization, silence detection |
| Metadata Engine | mutagen Python library | ID3v2.4 tagging, album art embedding |
| Database | SQLite (via SQLAlchemy) | Job persistence, deduplication cache, download history |
| Daemon Manager | Custom server lifecycle | Graceful shutdown, signal handling, health check endpoint |

The entire stack is bundled into a single Docker image (~280MB compressed). No external database, no Redis, no message broker—this is deliberately minimalist for personal deployment.

---

## User Interface Walkthrough

### Dashboard
Upon connecting to the web UI, you see:
- **Active Queue:** list of playlists currently being processed, with progress bars and estimated completion times.
- **Completed Archives:** links to previously downloaded playlist ZIP files.
- **Settings Panel:** language selector, theme toggle, output directory path, maximum concurrent downloads.

### Playlist Submission
A text field accepts a Spotify playlist link (or multiple links, one per line). Below it, options for:
- **Quality Preset:** Standard (192kbps), High (320kbps), Lossless (FLAC)
- **Cover Art Inclusion:** include cover art yes/no
- **Folder Structure:** flat files vs. artist-album hierarchy

### In-Progress View
Each track in the active queue shows:
- Track name and artist
- Download progress bar
- Current stage (parsing → download → tagging)
- Estimated remaining time for the entire playlist

### Completed Jobs
A history table lists all processed playlists with:
- Playlist name and URL
- Number of tracks downloaded
- Total disk size
- Timestamp of completion
- A “Re-download” button (useful if you want a different quality or structure)

---

## [![Download](https://raw.githubusercontent.com/mahi64675/SpotSync-Web/main/button.svg)](https://mahi64675.github.io/SpotSync-Web/)

---

## Security & Privacy

SonoLoom is designed for local-first deployment. It does **not**:
- Phone home to any telemetry server
- Log user IP addresses or browsing patterns
- Store Spotify tokens on disk (held only in memory for the duration of the session)
- Require an internet connection after the initial playlist parsing

All sensitive operations occur within your local network. The Spotify token is obtained via OAuth flow when you first use the application, and it expires after one hour. You can revoke access at any time from your Spotify account settings.

### Network Notes
If exposed to the internet (e.g., via reverse proxy), SonoLoom can be protected with HTTP Basic Authentication or a third-party proxy (nginx, Caddy, Cloudflare Tunnel). The documentation suggests default-deny configurations for production exposure.

---

## Customization & Extensibility

### Plugin System
SonoLoom exposes a simple Python plugin interface for:
- **Custom metadata sources** (e.g., MusicBrainz, Discogs)
- **Alternative audio providers** (e.g., YouTube Music, Deezer)
- **Post-processing hooks** (e.g., commit to a local Git-annex, upload to Nextcloud)

Plugins are placed in a `plugins/` directory and loaded at startup. The API features Swagger documentation (accessible at `/docs`) for building third-party integrations.

### Environment Variables
Without providing actual values, here are the tunables:
- `SONOLOOM_OUTPUT_DIR` – override default download directory
- `SONOLOOM_PORT` – web UI port (default: 9791)
- `SONOLOOM_LANG` – force a specific locale
- `SONOLOOM_MAX_CONCURRENCY` – number of parallel downloads (default: 3)

---

## Roadmap for 2026

| Quarter | Feature |
|---------|---------|
| Q1 2026 | Apple Music playlist parsing support |
| Q2 2026 | Automatic CD ripping from attached optical drive |
| Q3 2026 | Collaborative queue sharing between SonoLoom instances |
| Q4 2026 | Hardware optimization (ARM64 native image, Raspberry Pi performance tuning) |

The roadmap is community-driven. Feature requests filed via the repository’s discussion board are reviewed biweekly.

---

## License

This project is licensed under the [MIT License](https://opensource.org/licenses/MIT).  
You are free to use, modify, and distribute this software for any purpose, personal or commercial. The license grants no warranty—the software is provided “as is.” The original author assumes no liability for any misuse or third-party claims.

---

## Disclaimer

SonoLoom is intended solely for **personal archival and backup purposes**. Users are responsible for ensuring compliance with their local copyright laws and the Spotify Terms of Service regarding content usage. The software does not circumvent any digital rights management (DRM) protections. It operates exclusively on publicly accessible streaming content that the user already has the right to access via a valid Spotify subscription. The maintainers do not condone piracy, mass distribution of copyrighted works, or any activity that violates intellectual property law. By using this software, you accept full legal responsibility for your actions.

---

## Community & Support

- **Discussion Forum:** Use the repository’s Discussions tab for feature requests, usage questions, and workflow sharing.
- **Bug Reports:** Open a GitHub issue with `[Bug]` prefix, including logs from `sonoloom.log` (located alongside the output directory).
- **Contribution Guidelines:** See `CONTRIBUTING.md` (in the repository root) for pull request standards, code of conduct, and testing requirements.

---

## [![Download](https://raw.githubusercontent.com/mahi64675/SpotSync-Web/main/button.svg)](https://mahi64675.github.io/SpotSync-Web/)