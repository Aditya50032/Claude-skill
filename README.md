# Disk Cleanup v2

A Claude Code skill that finds what's actually using your disk space and safely reclaims storage.

Instead of blindly deleting files, Disk Cleanup v2 analyzes caches, build artifacts, stale dependencies, and development leftovers, then shows exactly what can be removed before taking any action.

## Features

* 🔍 Scans your machine for reclaimable storage
* 📦 Detects package manager caches (npm, pip, Go, etc.)
* 🗂 Finds stale `node_modules` directories
* 🐳 Identifies Docker leftovers
* 🛡 Dry-run by default (nothing is deleted automatically)
* ♻️ Quarantine-based recovery system
* 🔒 Safety-first path validation
* 📊 Reports estimated space savings before cleanup

---

## Requirements

* Claude Code
* Python 3.8+
* macOS or Linux

---

## Installation

### 1. Clone this repository

```bash
git clone https://github.com/Aditya50032/disk-cleanup-v2.git
```

### 2. Enter the repository

```bash
cd disk-cleanup-v2
```

### 3. Install the skill

Copy the skill folder into your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills
cp -R disk-cleanup-v2 ~/.claude/skills/
```

Alternatively, place it inside:

```text
.claude/skills/
```

if you only want it available for a specific project.

### 4. Restart Claude Code

Close and reopen Claude Code.

### 5. Verify installation

Inside Claude Code run:

```text
/skills
```

You should see **Disk Cleanup v2** in the installed skills list.

---

## Usage

You do not need to invoke the skill directly.

Simply describe your problem naturally:

```text
What's using all my disk space?
```

```text
My Mac is almost full. Help me clean it up.
```

```text
Find safe cleanup opportunities.
```

```text
Docker is taking too much storage.
```

The skill will:

1. Scan your machine
2. Generate a report
3. Estimate reclaimable storage
4. Ask for approval
5. Perform cleanup only after confirmation

---

## Example Output

```text
Reclaimable: 9.6 GB

SAFE
- Browser caches
- Pip cache
- Go build cache

REGENERABLE
- Stale node_modules

CAUTION
- Docker volumes
```

---

## Safety

Disk Cleanup v2 is designed to avoid destructive mistakes.

It will not delete:

* Git repositories
* Source code
* SSH keys
* Documents
* Desktop files
* Downloads
* Photos
* Music
* Videos
* Cloud storage folders

All cleanups are previewed before execution.

---

## Project Structure

```text
disk-cleanup-v2/
├── README.md
├── SKILL.md
├── scripts/
│   ├── scan.py
│   ├── clean.py
│   ├── restore.py
│   ├── quarantine.py
│   └── catalog.py
└── references/
    └── targets.md
```

---

## Contributing

Pull requests and suggestions are welcome.

Ideas for future improvements:

* Windows support
* Duplicate file detection
* Largest-file analysis
* Additional package manager integrations
* Storage usage visualization

---

## License

MIT License

---

Built by Aditya Pathak using Claude Code.
