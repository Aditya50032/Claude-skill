# Disk Cleanup v2

A cross-platform Claude Code skill that finds, analyzes, and safely reclaims disk space on **macOS, Linux, and Windows**.

Instead of blindly deleting files, Disk Cleanup v2 identifies caches, build artifacts, package manager leftovers, browser caches, stale dependencies, and other reclaimable storage. Every cleanup is previewed before execution, and nothing is removed without approval.

---

## Features

### Storage Analysis

* Scan your system for reclaimable disk space
* Identify the largest storage consumers
* Categorize cleanup opportunities by risk level
* Estimate space savings before cleanup

### Supported Targets

* npm cache
* pip cache
* yarn cache
* pnpm store
* Go build cache
* Go module cache
* Browser caches

  * Chrome
  * Brave
  * Edge
  * Firefox
* JetBrains IDE caches
* VS Code caches
* Build artifacts
* Stale node_modules directories
* Docker leftovers

### Safety Features

* Dry-run by default
* Approval required before cleanup
* Path validation system
* Protected directory enforcement
* Quarantine and restore support
* Detailed cleanup reports

---

## Supported Platforms

| Platform   | Status      |
| ---------- | ----------- |
| macOS      | ✅ Supported |
| Linux      | ✅ Supported |
| Windows 10 | ✅ Supported |
| Windows 11 | ✅ Supported |

---

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/Aditya50032/disk-cleanup-v2.git
```

### 2. Enter the project

```bash
cd disk-cleanup-v2
```

### 3. Install into Claude Code

#### Global Installation

```bash
mkdir -p ~/.claude/skills
cp -R disk-cleanup-v2 ~/.claude/skills/
```

#### Project-Specific Installation

Copy the folder into:

```text
.claude/skills/
```

inside your project.

### 4. Restart Claude Code

Restart Claude Code after installation.

### 5. Verify

Inside Claude Code:

```text
/skills
```

You should see:

```text
Disk Cleanup v2
```

---

## Usage

Simply describe your problem naturally.

### Examples

```text
What's using all my disk space?
```

```text
My Mac only has 5 GB left.
```

```text
Help me clean up my Windows laptop.
```

```text
Find safe cleanup opportunities.
```

```text
Docker is using too much storage.
```

```text
Analyze my system and generate a cleanup report.
```

The skill automatically:

1. Scans the system
2. Categorizes findings
3. Estimates recoverable space
4. Shows a cleanup plan
5. Waits for approval
6. Executes approved cleanup actions

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
- Build artifacts

CAUTION
- Docker volumes
```

---

## Risk Tiers

### SAFE

Automatically regenerated.

Examples:

* Browser caches
* pip cache
* npm cache
* Go cache

### REGENERABLE

Can be recreated but may require time or downloads.

Examples:

* node_modules
* build outputs
* package stores

### CAUTION

May contain important user data.

Examples:

* Docker volumes
* Xcode archives
* Database volumes

---

## Protected Paths

Disk Cleanup v2 will never remove:

### macOS/Linux

```text
~/Documents
~/Desktop
~/Downloads
~/Pictures
~/Music
~/Movies
~/.ssh
~/.gnupg
~/.aws
~/.kube
```

### Windows

```text
C:\Users\<user>\Documents
C:\Users\<user>\Desktop
C:\Users\<user>\Downloads
C:\Users\<user>\Pictures
C:\Users\<user>\Videos
C:\Users\<user>\Music
```

### Additional Protections

* Git repositories
* Source code
* SSH keys
* Environment files
* Cloud storage folders
* Symbolic links

---

## Repository Structure

```text
disk-cleanup-v2/
├── README.md
├── LICENSE
├── CONTRIBUTING.md
├── CHANGELOG.md
├── SKILL.md
├── scripts/
│   ├── catalog.py
│   ├── scan.py
│   ├── clean.py
│   ├── quarantine.py
│   └── restore.py
└── references/
    └── targets.md
```

---

## Roadmap

* [x] macOS support
* [x] Linux support
* [x] Windows support
* [x] Duplicate file detection
* [x] Largest file analyzer
* [x] Storage usage visualization
* [ ] HTML cleanup reports
* [x] Scheduled cleanup recommendations

---

## Contributing

Contributions are welcome.

Suggested improvements:

* Additional cache detectors
* Better reporting
* Performance improvements
* New package manager integrations
* Platform-specific enhancements

---

## License

MIT License

---

Built by Aditya Pathak using Claude Code.


---

Built by Aditya Pathak using Claude Code.
