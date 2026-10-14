# Fletch360 Mac hub: shared setup notes

Single source of truth for every chat/agent working on this. Update it whenever something changes. No secrets (passwords, keys, tokens) in this file.

## Goal
Mac mini as a server and alternate storage hub: files land in an inbox, then get archived, moved, or converted (audio options). Notion controls the agents via a job-queue database. Tailscale connects all devices; Termius is the SSH client.

## Machines
| Machine | Status |
|---|---|
| Mac mini (`Waynes-Mac-mini`, user `waynefletcher`) | Intel x86_64, macOS 12.7.6 Monterey. Homebrew unsupported here. Claude Code likely needs macOS 13+, so run agents from Windows PC instead. Remote Login (SSH) ON. Tailscale signed in. Terminal, VS Code, Termius installed. |
| Windows PC | Setting up Tailscale (needs cookies on in browser for sign-in). Use `winget`, not `brew`. |

## Storage
- SSD: `/Volumes/WD 1Tb Apple`. Journaled HFS+, about 686 GB free. Existing `SteamLibrary` there: do not touch.
- Hub path: `/Volumes/WD 1Tb Apple/hub/{inbox,processing,archive,converted}`. Folders may not exist yet (check with `ls`).

## Still to do
- [ ] ffmpeg on the Mac (no Homebrew: static binary from evermeet.cx, or MacPorts)
- [ ] ffmpeg on Windows if needed: `winget install Gyan.FFmpeg`
- [ ] Windows PC joined to Tailscale, SSH test to the mini
- [ ] Decide audio formats (e.g. WAV to FLAC archive, MP3/AAC share)
- [ ] Find out what "Brax*" is and how it uses Tailscale
- [ ] Notion job-queue database + Mac agent script
- [ ] launchd watcher on `inbox`

## Open questions
- Audio formats wanted?
- What is Brax* and where does it live?
- Volume rename (`WD 1Tb Apple` to `HUB`)? Optional.

## Hardware facts (from the Mac mini)
- Internal: APPLE SSD SM0256G, 250.79 GB APFS, about 50.7 GB free. SMART verified. Keep only scripts/config here.
- External: WD Elements SE (USB), 1 TB, Journaled HFS+, Apple Partition Map, about 686 GB free, "Ignore Ownership: Yes". Hub lives here. Scripts must check the volume is mounted before acting.

## Plan
- Now (Mac mini, macOS 12): SSH, Tailscale, SMB share, ffmpeg, launchd watcher, Notion job-queue agent. No Docker/Portainer (unsupported on this macOS).
- Later (new hardware): Docker + Portainer, containerize the watcher, add services (Syncthing, Uptime Kuma, Filebrowser). Migrate with git clone + rsync.
- Notion stays as the human-facing job queue/approvals; Portainer is only infra management.

## Kaboratech stack vs. this Mac (macOS 12 Intel, keep OS)
Phase 1, native on the mini (no Docker): Jellyfin (macOS 12+ per forum, verify), Navidrome (Intel macOS build, launchd, needs ffmpeg), Filebrowser Quantum (original File Browser archived 2026-09-01), ffmpeg, SMB share, Tailscale.
Netdata: macOS docs assume Homebrew, which is unavailable here; try source install or skip.
Phase 2, new hardware / Linux: OpenMediaVault (Debian only, replaces OS), Immich (Docker, 6 GB+ RAM, Linux strongly preferred), Pi-hole (Docker, loses per-client visibility on macOS), Portainer.
