# Fletch360 Mac hub: shared setup notes

Single source of truth for every chat/agent working on this. Update it whenever something changes. No secrets (passwords, keys, tokens) in this file.

## Goal
Mac mini as a server and alternate storage hub: files land in an inbox, then get archived, moved, or converted (audio options). Notion controls the agents via a job-queue database. Tailscale connects all devices; Termius is the SSH client.

## Machines
| Machine | Status |
|---|---|
| Mac mini (`Waynes-Mac-mini`, user `waynefletcher`) | Intel (Homebrew refuses to install). Remote Login (SSH) ON. Tailscale signed in. Terminal, VS Code, Termius installed. |
| Windows PC | Setting up Tailscale (needs cookies on in browser for sign-in). Use `winget`, not `brew`. |

## Storage
- SSD: `/Volumes/WD 1Tb Apple`. Journaled HFS+, about 686 GB free. Existing `SteamLibrary` there: do not touch.
- Hub path: `/Volumes/WD 1Tb Apple/hub/{inbox,processing,archive,converted}`. Folders may not exist yet (check with `ls`).

## Still to do
- [ ] ffmpeg on the Mac (no Homebrew: MacPorts or static binary)
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
