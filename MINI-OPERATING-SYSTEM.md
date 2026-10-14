# Mac mini "operating system": human-first principles, loops, use cases, tests

DRAFT, written 2026-10-14. Nothing here has run. Source: the `Fletch360/ai-stack` repo (read from a cloud session: CHARTER.md,
HOW-WE-WORK.md, PLAN-agent-charter.md, home-hub/PLAN.md, guide/SKILL.md, FAILURES.md, handoff/SKILL.md, rubric.json, stop_gate.py,
sample-library/PLAN.md, TOOLS-EVALUATED.md). The Windows scripts (PowerShell) were not run or tested. The Jev docs were not read
(docs.typesafe.ai is blocked from the cloud container), so Jev behaviour below is design, not fact.

This file describes the Mini as it is now: Intel, macOS 12.7.6, macOS kept, no Docker, no Homebrew.

## 1. Corrections to the ai-stack plans (found by comparing them with what the Mini actually reported)
| Plan says | Mini reported | Consequence |
|---|---|---|
| "Mac mini 2012", wipe to OpenMediaVault/Linux | Runs macOS 12.7.6 (a 2012 mini cannot; it is a later model) | Model/RAM numbers in the plan are unverified. Run `system_profiler SPHardwareDataType` and record it. |
| Claude Code on the Mac after the Linux install | macOS 12 kept; Claude Code needs macOS 13+ (the plan's own research) | Claude cannot run on the Mini while macOS 12 stays. Use the plan's own fallback: Claude runs on the PC (or cloud), the Mini does storage, scripts, Tailscale, ffmpeg. |
| WD drive = 314 GB of music, HFS+, mount read-only | 1 TB HFS+ drive, about 292 GiB (about 314 GB) used; top level shows only `SteamLibrary` | Same size, different label. Check what is inside `SteamLibrary` before calling it music. |
| Hub folders at `/srv/ai/...`, Linux users, `chattr +a` | macOS: `/Users/...`, `/Volumes/...` | Needs macOS equivalents (section 4). |
| Docker for agents, Open WebUI, Portainer | No usable Docker on macOS 12 | Containers wait for the new hardware. |

## 2. Human-first principles (taken from the build)
1. **The human owns the rules.** Charter, denylist, rubric and tier changes are human-only. Agents may propose, never edit.
2. **Start at propose-only (T0), earn trust with evidence.** Agents move up only when the human edits a file; scores only propose.
3. **Claims need evidence.** "Done" means a command ran and its output was attached. Self-reported success scores zero. "Unverified" is a legal status.
4. **The model proposes, code checks and acts.** Model output never becomes a command, path or send without a fixed rule in between.
5. **Rules need enforcers.** A rule with nothing enforcing it is marked ADVISORY and shown to the human, not trusted.
6. **Ratchet.** Judgement answers (Jev/LLM) can only tighten restrictions. Loosening is human-only.
7. **Private data stays on local models.** Private-looking tasks get local models, no outside send.
8. **Anything outward asks first.** Sends, posts, uploads, registrations, spending, legal: always the human.
9. **One front door, one notebook.** One place to work, one file for state (STATE.md), one queue (PENDING.md). Other windows only answer questions.
10. **Plain-language guidance.** The user is not a developer: say WHERE (which program), WHAT, and what success looks like. Commands in clean blocks, one purpose each.
11. **Cheapest verified path first; do not repeat a step that did not work, change the method.**
12. **Failures are logged and turned into permanent tests** (FAILURES.md, then bench scenarios).

## 3. The five skills and the loops
The rubric rewards five skills, each by a checkable event:
- **taste** (caught a defect, a lesson got promoted), **agency** (safe first attempt on record), **focus** (task with zero charter denials),
- **resourcefulness** (probed before saying "can't"), **follow-through** (verified done, accepted handoff).

Loops to build into the Mini:
| Loop | Cycle | Stop / safety condition |
|---|---|---|
| Session | read STATE.md and PENDING.md, state NEXT ACTION in one line, work, write Done/Now/Next, commit | no reply ends with uncommitted or unpushed work |
| Evidence | do the work, run the check, attach output, a separate verifier re-runs it | claim with no matching re-run scores negative |
| Try-first | before "can't": run a probe (status command, listing, search), quote the result | "can't" with no tool call is blocked |
| Round budget | after 3 exchanges with no verified result: state the blocker and the fastest unblock | no fourth partial round |
| Learning | attempt, verify, score, human verdict, add as a bench test, candidate lesson, human promotes | lessons need owner, evidence, review-by date; stale ones decay |
| Memory ladder | working, candidate, trusted | only the human promotes anything that changes rules or tiers |
| Handoff | write the packet so a session with zero memory can continue | new session starts from the default branch; unmerged work is invisible |
| Preflight | typed questions about a task (private? money/legal? outward? risk?) before it runs | answers can only restrict |

## 4. What changes on the Mini (macOS 12, no Docker)
| Idea in the plan (Linux) | macOS equivalent (design, untested) |
|---|---|
| One Linux user per agent | Separate standard macOS user accounts per role (human, keeper, agent); agents never get admin |
| `/srv/ai/charter` read-only to agents | Owned by the human account, mode 0444/0555 for others |
| Append-only ledger (`chattr +a`) | `chflags uappend` on the ledger file (verify behaviour on HFS+; the external drive ignores ownership, so keep the ledger on the internal APFS drive) |
| Write-only drop box `incoming/` | Folder with an ACL allowing create but not read/delete for agents (needs a test on the real Mini) |
| systemd services | `launchd` plists |
| Docker containers | Plain scripts under `launchd`; containers come with the new hardware |
| Claude Code on the Mac | Not available on macOS 12; the PC/cloud session drives the Mini over Tailscale/SSH |
| Ignore Ownership on the external SSD | Turn it off for any zone that needs permissions to mean something |

## 5. Zones (from the ai-stack design) mapped to the Mini
`private/`, `business/`, `shared/`, `backups/`, `incoming/`. The new-file watcher works only on `incoming/`. Audio that already sits in the vaults stays
where it is: the watcher never moves, renames or converts it.

## 6. Use cases and test conditions
Each test runs on a throwaway folder with fake files first. "Pass" is what must be seen.
| # | Use case | Pass condition |
|---|---|---|
| U1 | New audio file lands in `incoming/` | Logged with sha256; Jev (or the stand-in) returns type + confidence; routed to convert, archive or hold; nothing in a vault changed |
| U2 | Same file dropped twice | Second one flagged duplicate by hash, no second conversion, nothing deleted |
| U3 | Near-duplicate (same sample, different processing) | Listed for the human; never auto-deleted |
| U4 | Unknown or blocked licence/provenance | Quarantined, never reaches `ready/` |
| U5 | Looks private (personal, health, finance, family) | Local model only, no outward send, held for the human |
| U6 | Needs money, legal or an outside send | Always asks the human |
| U7 | Low confidence or ambiguous Jev answer | Goes to the LLM stage or "Needs review"; the LLM answer can only tighten |
| U8 | Jev or the network is unreachable | Item waits as "Needs review"; no guessing; no loss |
| U9 | External SSD unmounted mid-run | Watcher stops and logs; nothing is written to the internal drive |
| U10 | Conversion output | Output exists, plays, duration within tolerance of the source, hash recorded; original kept until verified |
| U11 | File already in a vault | Negative test: untouched, same hash before and after |
| U12 | Agent tries to edit a protected file | Denied and logged as a charter denial |
| U13 | Claim of "done" with no evidence | Scored negative; push or promotion blocked |
| U14 | Disk below a set free-space limit | Watcher pauses and warns (internal drive has about 50 GB free) |
Refine from results: each failure becomes a permanent test; thresholds (confidence cut-off, duration tolerance, free-space limit) are set by the human after seeing real runs.

## 7. LLM stage rules
- Order: fixed rules, then Jev answers typed questions (yes/no probability or pick-one), then the LLM only for low-confidence or ambiguous items.
- Confidence cut-off is a number the human sets after the first real runs (not guessed here).
- LLM output is data, not a command. It can add restrictions or a note; only fixed code acts.
- Private-looking items never go to a cloud model.
- Everything logged to the ledger with the question, answer, confidence and which rule acted.
- Budget: a cap on LLM calls per day, with "Needs review" as the overflow.

## 8. Recommendations for managing the information
1. **One notebook, in the private repo.** Move this plan and `SETUP-NOTES.md` into `ai-stack/home-hub/` (private) and keep only STATE/PENDING as the live record. Do not keep parallel notes in other repos.
2. **Do not put machine names, usernames, IPs, key-handling or private-data details in a public repo.** `Fletch360/Fletch360` is public; `SETUP-NOTES.md` there names the Mini and its user. Decide whether to scrub it.
3. **Facts get a status:** verified (with the pasted output), unverified, or advisory. Keep the date.
4. **Ledger is append-only and copied off the machine weekly** so tampering shows.
5. **Review cadence:** weekly human check of the score summary, the ADVISORY rules, and a sample of verified items.
6. **Lessons expire** unless reconfirmed; retire them in the open instead of silently deleting.
7. **Keep the Windows build and the Mini plan reconciled** with a short conflict register, as home-hub/PLAN.md does (C1-C12), and close items with evidence.
8. **Backups:** the 1 TB external is the only large store; keep a second copy of anything that matters before any conversion or archive step.

## 9. Open decisions for the human
1. Where this file lives (private repo recommended) and whether to scrub `SETUP-NOTES.md` from the public repo.
2. Audio formats for convert/archive/share.
3. What is inside `SteamLibrary` on the external drive (music or games).
4. The Mini's exact model and RAM (`system_profiler`).
5. Confidence cut-off, free-space limit and LLM daily cap.
6. Whether Jev runs as a hosted API (needs the Jev docs read from the PC).
