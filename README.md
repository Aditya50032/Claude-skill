# disk-cleanup

A [Claude Code](https://claude.com/claude-code) skill that finds and safely reclaims disk space.

Ask Claude why your disk is full. It measures what's actually consuming space — package manager caches, stale `node_modules`, Docker leftovers, Xcode data — sorts it by how much it costs you to lose, shows you the numbers, and deletes only what you approve.

```
Reclaimable: 68 GB   (free space would go 23 GB -> 91 GB)

SAFE (41 GB)          npm cache 12 GB · ~/Library/Caches 21 GB · pip cache 8 GB
REGENERABLE (24 GB)   34 stale node_modules 19 GB (reinstall: npm install)
CAUTION (3 GB)        2 unused Docker volumes — may hold database data
```

---

## Install

Download `disk-cleanup.skill` and unpack it into your skills directory:

```bash
unzip disk-cleanup.skill -d ~/.claude/skills/
```

Use `.claude/skills/` inside a project instead if you only want it there. Restart Claude Code, then confirm it registered:

```bash
/skills
```

Requires Python 3.8+ and `du`, both standard on macOS and Linux. No third-party packages.

---

## Usage

You don't invoke the skill by name. Describe the problem and it triggers:

> my mac is at 4gb free, help
>
> what's eating all my disk space?
>
> docker is using like 80gb, can you clean it up
>
> "no space left on device" — I can't even save files

Claude scans, reports, and waits for your approval before deleting anything. Reply with what you accept — *"do the safe and regenerable ones"*, or *"just npm and pip"* — and it cleans exactly that.

### Running the scripts yourself

The scan is read-only and safe to run any time:

```bash
cd ~/.claude/skills/disk-cleanup

python3 scripts/scan.py                            # start here
python3 scripts/scan.py --paths                    # list individual directories
python3 scripts/scan.py --projects ~/work ~/code   # if your repos live elsewhere
python3 scripts/scan.py --stale-days 90            # only projects idle 90+ days
```

To clean manually, write a plan first and act on it:

```bash
python3 scripts/scan.py --json plan.json
python3 scripts/clean.py --plan plan.json --tiers safe                      # dry run
python3 scripts/clean.py --plan plan.json --tiers safe,regenerable --confirm
python3 scripts/clean.py --plan plan.json --ids npm-cache,pip-cache --confirm
```

Without `--confirm`, `clean.py` only prints what it *would* remove. Every real run appends each path and size to `~/.disk-cleanup/cleanup-<timestamp>.log` — that log is how you find out what went if something turns out to have been needed.

### Options

**`scan.py`**

| Flag | Effect |
|---|---|
| `--projects PATH...` | Roots to search for build artifacts (default: `~/code`, `~/projects`, `~/src`, `~/dev`, `~/work`, `~/repos`, `~/Documents/GitHub`) |
| `--stale-days N` | Only report artifacts whose project is untouched this long (default 30) |
| `--json FILE` | Write a machine-readable plan for `clean.py` |
| `--paths` | List individual paths under each target |
| `--no-projects` | Skip the project walk; caches only |

**`clean.py`**

| Flag | Effect |
|---|---|
| `--plan FILE` | Plan produced by `scan.py --json` (required) |
| `--tiers` | Comma-separated: `safe,regenerable` |
| `--ids` | Comma-separated target ids, e.g. `npm-cache,xcode-derived` |
| `--exclude` | Target ids to skip |
| `--confirm` | Actually delete — **omit this for a dry run** |
| `--allow-sudo` | Permit targets whose purge command needs sudo |

---

## Risk tiers

Every target lands in one of three tiers, which is what the approval step is really about.

| Tier | Meaning | Example |
|---|---|---|
| **`safe`** | Regenerates automatically. Costs you a slower next command, nothing else. | npm cache, `__pycache__` |
| **`regenerable`** | Comes back, but only by running something. Real time and bandwidth. | `node_modules` → `npm install` |
| **`caution`** | May hold data with no other copy, or takes very long to recreate. | Docker volumes, Xcode Archives |

`caution` items can't be swept by tier. `clean.py --tiers caution` is refused outright; you have to name each one with `--ids`. A Docker volume may be the only copy of your local database, and an Xcode Archive is what lets you symbolicate crash reports from a shipped release.

---

## How the safety works

The guardrails live in code, not in prose instructions to the model. A skill that just says "be careful" is one bad inference away from `rm -rf ~/Documents`.

`clean.py` re-validates every path against a denylist immediately before deleting it, independently of whatever the plan file claims. It's also an allowlist: a path has to be a known artifact name or a catalogued cache path, or it's refused regardless of what asked for it.

**Never deleted, under any circumstances:**

- Source files, and any directory containing `.git`
- `.env`, `id_rsa`, `id_ed25519`, `.netrc`, `.pgpass`
- `~/.ssh`, `~/.gnupg`, `~/.aws`, `~/.kube`
- `~/Documents`, `~/Desktop`, `~/Downloads`, `~/Pictures`, `~/Music`, `~/Movies`
- Dropbox, OneDrive, Google Drive, iCloud Drive folders
- Symlinks — deleting one would delete through to its target
- `/`, `/usr`, `/etc`, `/var`, `/home`, `$HOME`, and anything under three components deep

One deliberate exception: a directory whose own *final* component is a known artifact stays deletable inside an otherwise protected location, because `~/Documents/GitHub/project/node_modules` is genuinely just `node_modules`. Being merely *inside* `Documents` never qualifies.

### Verified against attack

The guardrails were tested against a fixture containing real user data:

- **25 path-validator cases** — `/`, `/etc`, `$HOME`, `~/Documents`, `~/Downloads`, `~/.ssh/id_rsa`, `.env`, git repo roots, source directories, and a symlink named `node_modules` pointing at `~/Documents`. All denied; genuine artifacts allowed.
- **Tampered-plan attack** — the plan file was hand-edited to inject `~/Documents/thesis`, `~/Downloads`, `~/.ssh`, and `/etc` into an already-approved target. All seven refused with reasons, while the legitimate `node_modules` was still cleaned.
- **Dry-run default and tier sweeps** — `--confirm` omitted deletes nothing; `--tiers caution` is refused; unknown tier names error out.

---

## Design notes

**Prefer the tool's own cleaner.** Where a package manager ships a purge command, `clean.py` runs that instead of `rm -rf` — `npm cache clean --force`, `brew cleanup -s`, `uv cache clean`. Package managers keep index files alongside cached blobs, and removing the directory underneath leaves the tool believing it still has content it doesn't.

**`target/` and `build/` need a sibling manifest.** Both names are far too common as ordinary source directories to delete on the name alone. A `target/` only counts as a Rust build output if `Cargo.toml` sits next to it; `build/` needs `CMakeLists.txt`.

**Staleness is a safety filter, not a nag.** Anything touched in the last 30 days is skipped, so an active project's `node_modules` never gets proposed — deleting it mid-work costs an immediate reinstall and breaks a running dev server. Drop to `--stale-days 7` if you're genuinely desperate.

**Docker gets special handling.** `docker system prune` is fine with approval. `docker system prune -a` re-pulls every image, which can be hours on a slow link, so the cost gets stated explicitly. `docker volume prune` is never bulk-approved.

**If the space is your own files, it says so.** When the biggest directory is photos, video projects, or datasets, the honest answer is "your data is the thing taking up space" — not a creative reinterpretation of what counts as a cache.

---

## Layout

```
disk-cleanup/
├── SKILL.md                 workflow, tiers, and judgment calls
├── scripts/
│   ├── catalog.py           target definitions + validate_path() — the safety core
│   ├── scan.py              read-only measurement and reporting
│   └── clean.py             execution, dry-run by default
└── references/
    └── targets.md           full per-platform catalog with regeneration costs
```

---

## Extending it

Add a target to `TARGETS` in `scripts/catalog.py`:

```python
_t("mytool-cache", "MyTool cache", SAFE, ["~/.cache/mytool"],
   platforms=("darwin", "linux"),
   regen="repopulates on next mytool build",
   purge_cmd="mytool cache clear")     # optional, preferred over rm -rf
```

Then check three things:

1. **Tier it honestly.** If losing it costs a rebuild it's `regenerable`, not `safe`. If it could hold data with no other copy it's `caution`, however large and tempting it looks.
2. **Make sure `validate_path()` accepts it.** Catalogued paths pass automatically. A new artifact *directory name* also has to go in `ARTIFACT_BASENAMES`, or the validator refuses it at delete time even though the scanner found it.
3. **Document it** in `references/targets.md`.

---

## Caveats

Tested on Linux and written for macOS and Linux. Windows isn't supported — WSL works, since it's Linux underneath.

Sizes come from `du`, which reports disk usage rather than apparent size. On filesystems with compression or deduplication (APFS, ZFS, btrfs), reclaimed space can differ from the estimate.

`scan.py` only reports targets over 1 MB (5 MB for project artifacts), so a long tail of small caches won't appear. That's deliberate — a report you can read beats a report that's complete.

---

## Disclaimer

Deleting files is irreversible. The safeguards here are thorough and tested, but back up anything you can't afford to lose before running cleanup tools — this one or any other.
