# Apache Smart Tuner

Apache Smart Tuner is a lightweight Bash utility that inspects an Apache host and proposes right-sized MPM settings based on live resource data. It focuses on safety and visibility so system administrators can apply concurrency tuning confidently on both cPanel and generic Apache deployments.

## Why it helps
- **Resource-aware tuning:** Uses RAM, CPU cores, and observed Apache process sizes to set conservative but efficient limits.
- **Tiered profiles:** Automatically selects LOW, LOW-MID, MID, or HIGH tiers to cap concurrency without starving smaller hosts.
- **cPanel integration:** Writes to standard include locations and triggers `rebuildhttpdconf` when needed.
- **Safe changes:** Creates backups, validates configs with `configtest`, and rolls back on failure.
- **Clear diffs:** Prints a side-by-side view of current versus proposed values before applying changes.

## Quick start
- View recommendations (default):
  ```bash
  ./apache-tuner
  ```
- Show where MPM directives are defined:
  ```bash
  ./apache-tuner --locate
  ```
- Apply tuned settings (requires root):
  ```bash
  sudo ./apache-tuner --apply
  ```
- Check the current tool version:
  ```bash
  ./apache-tuner --version
  ```

## Versioning
Apache Smart Tuner now follows semantic versioning. The current release is **1.16.0**, reflecting the sixteenth iteration of the tuner within a structured MAJOR.MINOR.PATCH scheme.

## What's new in 1.16.0
- Detects whether Apache is currently running and reports the observed process count during analysis.
- Skips reloads when Apache is stopped while still applying configuration safely, prompting an administrator to start the service manually.

## License and attribution
Apache Smart Tuner is licensed under the GPL-3.0-or-later. Authored by **Ryan MacDonald <ryan@rfxn.com>**.
