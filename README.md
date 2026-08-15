# Disk Cleanup v2

A Claude Code skill that finds and safely reclaims disk space.

## Features

- Detects package manager caches
- Finds stale node_modules directories
- Identifies Docker leftovers
- Reports reclaimable space before cleanup
- Dry-run by default
- Quarantine-based recovery system
- Restore deleted items for up to 7 days
- Safety-first path validation

## Example

Reclaimable: 68 GB

SAFE (41 GB)
- npm cache
- pip cache
- browser caches

REGENERABLE (24 GB)
- stale node_modules

CAUTION (3 GB)
- Docker volumes

## Installation

```bash
git clone https://github.com/YOUR_USERNAME/disk-cleanup-v2.git
