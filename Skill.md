---
name: disk-cleanup-v2
description: Find and safely reclaim disk space on macOS, Linux, or Windows 10/11 by measuring what is actually consuming it — package manager caches, build artifacts, stale node_modules, Docker leftovers, Xcode data, browser caches, system caches — then removing only what the user explicitly approves, into a restorable quarantine rather than deleting outright. Use this skill whenever the user mentions running out of disk space, a full or nearly-full drive, "no space left on device" errors, wanting to free up / clear / reclaim storage, cleaning caches or node_modules, Docker eating their disk, or asks what is taking up space on their machine — even if they never use the word "clean".
---

# Disk Cleanup (v2.1 — quarantine + Windows)

Reclaim disk space without ever destroying something the user cannot get back. Works on macOS, Linux, and Windows 10/11 — the platform is detected automatically; there is nothing to configure.

**What changed from v1:** approved items are *moved* into a quarantine directory instead of being unlinked, and stay restorable for 7 days. Cleanup is now reversible. The catch is that space isn't actually reclaimed until the quarantine is purged — see "The quarantine tradeoff" below, because it changes what you tell the user.

**What changed in v2.1:** Windows 10/11 support, with its own cache catalog (Temp, browser caches, VS Code, and the same package-manager caches as macOS/Linux) and its own path protections (`Documents`, `Desktop`, `Downloads`, `OneDrive`, case-insensitive matching, `C:\Windows` and friends). macOS and Linux behaviour is byte-for-byte unchanged — see `CHANGELOG.md`.

## The core rule

**Scan, report, confirm, then delete — in that order, every time.**

Never delete anything in the same turn you discovered it. The user sees sizes and a category list first, and says yes to specific categories. This is not bureaucratic caution: the difference between `node_modules` (regenerates with one command) and a Docker volume (may hold the only copy of a database) is invisible from the filesystem alone, and getting it wrong is unrecoverable.

If the user says "just clean everything, don't ask me" — still run the scan, still show the report, but you may then proceed with `safe` tier only and tell them what you skipped and why. Never auto-approve `caution` tier.

## Workflow

### 1. Scan (read-only, always safe to run)

```bash
python3 scripts/scan.py                          # home dir + common project roots
python3 scripts/scan.py --projects ~/code ~/work # specific project roots
python3 scripts/scan.py --stale-days 90          # only artifacts untouched for 90+ days
python3 scripts/scan.py --json plan.json         # machine-readable, feeds clean.py
```

`scan.py` never deletes, never writes outside the file you name with `--json`. It reports total disk usage, then every reclaimable target it found, grouped by risk tier with a size for each.

Start with a plain `scan.py`. Only reach for `--projects` if the default roots miss where they keep code.

### 2. Report to the user

Lead with the number that matters: how much is reclaimable, against how much is free now. Then group by tier, biggest first. Keep it short — a wall of paths is not a report.

Something like:

> You've got 23 GB free. I found **68 GB** reclaimable:
>
> **Safe (41 GB)** — regenerates automatically, no action needed from you
> - npm + pnpm caches — 12 GB
> - pip / uv caches — 8 GB
> - `~/Library/Caches` — 21 GB
>
> **Regenerable (24 GB)** — comes back, but costs a reinstall or rebuild
> - 34 stale `node_modules` untouched 90+ days — 19 GB (reinstall: `npm install`)
> - Rust `target/` dirs — 5 GB (rebuild: `cargo build`)
>
> **Caution (3 GB)** — check these yourself before I touch them
> - 2 unused Docker volumes — 3 GB (may hold database data)
>
> Want me to take the safe and regenerable ones? That's 65 GB.

Then stop and wait. Do not proceed on an ambiguous answer.

### 3. Clean (only after explicit approval)

```bash
python3 scripts/clean.py --plan plan.json --tiers safe                     # dry run
python3 scripts/clean.py --plan plan.json --tiers safe,regenerable --confirm
python3 scripts/clean.py --plan plan.json --ids npm-cache --confirm --retention-days 14
python3 scripts/clean.py --plan plan.json --tiers safe --confirm --no-quarantine   # disk full: delete now
```

`clean.py` dry-runs unless `--confirm` is passed. It refuses any path that isn't in the plan the scanner produced, and independently re-validates every path against its own denylist before unlinking — so a hand-edited plan file still can't make it delete `~/Documents`.

Every run appends to `~/.disk-cleanup/cleanup-<timestamp>.log` with each path and its size. When something turns out to have been needed, that log is how you find out what went.

### 4. Manage the quarantine

```bash
python3 scripts/restore.py --list             # batches, sizes, expiry dates
python3 scripts/restore.py --list BATCH       # what's inside one batch
python3 scripts/restore.py --restore BATCH    # put it all back
python3 scripts/restore.py --restore BATCH --only /path/to/node_modules
python3 scripts/restore.py --purge BATCH      # reclaim the space now, permanently
python3 scripts/restore.py --purge            # purge everything past retention
```

Expired batches are purged automatically at the start of the next `clean.py --confirm` run, so the quarantine does not grow without bound.

Restoring never overwrites. If something has recreated the original path in the meantime — a fresh `npm install` between cleanup and restore — the recovered copy lands at `<path>.restored` and the newer one is left alone.

### 5. Confirm the result

Report actual reclaimed space (`clean.py` prints it) and re-check free space. If a category failed — permissions, file in use — say so plainly rather than reporting a number that didn't materialise.

## The quarantine tradeoff

Quarantine moves files rather than deleting them, which means **the space is not free yet**. Report this honestly — a user who cleaned 60 GB and sees no change in free space will reasonably think the tool is broken.

Say it plainly: *"60 GB moved to quarantine, restorable for 7 days. Free space is unchanged until it's purged — I can purge now if you need the space immediately."*

Two situations call for `--no-quarantine`:

- **The disk is genuinely full.** There may not be room to hold anything, and the user needs space now, not an undo option.
- **The reclaimed amount is huge relative to free space.** Holding 200 GB for a week on a drive with 40 GB free helps nobody.

Two things quarantine cannot cover, so say so rather than implying everything is reversible:

- **Command-driven targets.** Docker prunes, `journalctl --vacuum`, `simctl delete` — an external tool does the deleting and there is nothing to intercept. These are permanent.
- **Cross-filesystem items.** Quarantine is a rename within one filesystem. A path on another mount is skipped with a note, because copy-then-delete would briefly double disk usage — the worst possible behaviour mid-cleanup. Clean those with `--no-quarantine`.

Where a target has both a purge command and known paths, quarantine mode prefers moving the paths, since a reversible move beats an irreversible external purge. Pass `--no-quarantine` to get v1's behaviour of running the tool's own cleaner.

## Risk tiers

The scanner assigns these. Understand them before overriding anything.

| Tier | Meaning | Approval needed |
|---|---|---|
| `safe` | Regenerates automatically on next use. Deleting it costs nothing but a slower next command. | Confirm, but low stakes |
| `regenerable` | Comes back, but only by running something — `npm install`, `cargo build`, a Docker rebuild. Costs time and bandwidth. | Explicit yes, and tell them the rebuild cost |
| `caution` | May contain data with no other copy, or takes a very long time to recreate. Docker volumes, Xcode device support, Simulator runtimes. | Explicit yes **per item**, after saying what could be lost |

Nothing outside these tiers is ever a deletion candidate. The scanner will not surface source code, `.git`, `.env` files, databases, Documents, Desktop, or Downloads — regardless of size. If the user's disk is full of their own files, the honest answer is "your data is the thing taking up space", not a creative reinterpretation of what counts as a cache.

## Windows notes

Browser and editor caches (Chrome, Brave, Edge, VS Code) can be **locked by a running process**. If a delete fails with a permission error on Windows, that's almost always why — say so and suggest closing the application, don't imply something went wrong with the tool.

Several Windows targets need a real profile to test against — there is no `du` or POSIX home directory to fall back on. `tests/test_paths.py` covers the path *policy* (what would or wouldn't be allowed) without needing a Windows machine at all; run it on any platform.

## Prefer the tool's own cleaner

Where a package manager ships a purge command, run that instead of deleting its cache directory — it keeps internal indexes consistent. The scanner records this in each target's `purge_cmd`, and `clean.py` uses it automatically:

| Instead of `rm -rf` | Run |
|---|---|
| `~/.npm/_cacache` | `npm cache clean --force` |
| `~/Library/Caches/Homebrew` | `brew cleanup -s` |
| Docker leftovers | `docker system prune` (never add `--volumes` without per-item approval) |
| `~/.cache/uv` | `uv cache clean` |

## Docker deserves its own paragraph

Docker is usually the single biggest win and the single biggest hazard.

- `docker system prune` — dangling images, stopped containers, build cache. Regenerable. Fine with approval.
- `docker system prune -a` — **also removes every image not attached to a running container.** On a slow connection this is hours of re-pulling. Flag that cost explicitly.
- `docker volume prune` — **can destroy database data.** Treat every volume as `caution`. List them individually with `docker volume ls` and make the user approve each. Never fold volumes into a bulk approval.

## Reading the situation

**"No space left on device" right now.** They're blocked. Go straight for the biggest `safe` wins, get them unblocked in one round, then offer a fuller pass. Don't make someone who can't save a file sit through a full report.

**A server or CI box, not a laptop.** Check `journalctl --disk-usage`, `/var/log`, and old kernels (`apt autoremove`). Skip anything under `~/Library` — that's macOS. The scanner detects the platform, but sanity-check its output against where you actually are.

**They ask "what's taking up space?"** That's a scan request, not a cleanup request. Report and stop. Don't propose deletions unless they ask.

**The big directory is their own data.** Photos, video projects, datasets, VM images. Say so. Offer to identify what's largest so they can decide — archiving or moving to external storage is their call, not a cleanup action you take.

## Things that are never cleanup targets

Source files · `.git` directories and their objects · `.env`, credentials, SSH keys, keychains · databases and their data directories · `~/Documents`, `~/Desktop`, `~/Downloads`, `~/Pictures` · anything in Trash the user hasn't asked about · another user's home directory · any path requiring `sudo` unless the user explicitly asks and understands why.

`~/Downloads` in particular is a standing temptation — it's often huge and looks disposable. It is user data. Report its size if asked; never delete from it.

## Reference

`references/targets.md` — the full per-platform catalog of what the scanner looks for, what tier each target is, and how it regenerates. Read it when you need to explain a specific target to the user, or when adding a new one.
