# Apache Smart Tuner

Apache Smart Tuner is a Bash-based capacity planner for Apache HTTP Server that favors safety and clarity for busy shared hosting stacks (cPanel/WHM and generic Apache alike). The tuner inspects live system resources, applies tiered guardrails, and produces right-sized MPM settings that respect PHP-heavy mixed workloads without risking RAM exhaustion.

## What it does
- **Resource- and PHP-aware tuning:** Uses total RAM, CPU cores, observed httpd RSS, and PHP-friendly CPU caps to calculate safe concurrency limits per MPM.
- **Tiered profiles:** LOW, LOW-MID, MID, and HIGH profiles automatically cap ServerLimit/MaxRequestWorkers (or threads) for a wide range of footprints.
- **Cautious floors and caps:** Enforces sensible minimums (128 workers/threads) while honoring tier ceilings to avoid runaway prefork deployments.
- **Clear visibility:** Prints current versus proposed settings, emits an annotated config block, and now supports machine-readable JSON output for automation.
- **Safe application path:** Creates timestamped backups, scrubs legacy MPM blocks, validates with `configtest`, and reloads (or optionally skips reload) only after a clean validation.
- **cPanel/WHM integration:** Prefers `pre_virtualhost_global.conf` when present, falls back to `pre_main_global.conf`, and triggers `/scripts/rebuildhttpdconf` when applying changes.
- **Operator control:** Override the RAM budget percentage when you need a custom envelope and redirect or disable on-disk logging per run.

## Usage
Download or copy `apache-tuner` to a host where Apache is installed and make it executable:

```bash
chmod +x apache-tuner
```

### Modes
- **Analyze (default):** Inspect RAM/CPU/process data and print the proposed tuning block.
- **Locate:** Show which Apache config files currently define MPM or concurrency directives.
- **Apply:** Safely write the tuned block to the appropriate include file (root required).

### Flags
```
./apache-tuner [--analyze] [--locate] [--apply] [--json] [--no-reload] [--version] [--budget <0.x>] [--log-file <path|none>]
```
- `--json`: Return analysis as JSON (for monitoring/CM pipelines). Only valid with analyze mode.
- `--no-reload`: When combined with `--apply`, write the config but skip the reload/restart.
- `--version`: Print the current Apache Smart Tuner release.
- `--budget`: Override the tier-derived Apache RAM budget with a decimal percentage (e.g., `0.40`).
- `--log-file`: Send tuner logs to a custom file or `none` to disable filesystem logging.

### Examples
Run a standard analysis:

```bash
./apache-tuner
```

Produce JSON suitable for automation or dashboards:

```bash
./apache-tuner --json
```

Inspect where existing MPM settings live:

```bash
./apache-tuner --locate
```

Apply tuned values on a cPanel server while deferring the reload to a maintenance window:

```bash
sudo ./apache-tuner --apply --no-reload
```

## What to expect
Text analysis output includes environment detection, current vs proposed values, and the ready-to-paste block:

```
------------------------------------------------
 Apache Smart Tuner Analysis
------------------------------------------------
Detected MPM:           prefork
Tier:                   LOW-MID
Total RAM:              4096 MB
CPU Cores:              4
Apache running:         yes (180 processes observed)
Apache RAM Budget:      1434 MB (0.35 of total; source: tier)
Avg httpd proc size:    12 MB
------------------------------------------------
Current vs Proposed (prefork) (current => proposed):
  ServerLimit           256          => 384
  MaxRequestWorkers     256          => 384
  StartServers          5            => 8
  MinSpareServers       5            => 8
  MaxSpareServers       10           => 16
  MaxConnectionsPerChild 0           => 4000
------------------------------------------------
# BEGIN APACHE_SMART_TUNER
# Apache Smart Tuner v1.17.0 (Tier: LOW-MID, MPM: prefork)
Timeout 120
KeepAlive On
MaxKeepAliveRequests 100
KeepAliveTimeout 5

<IfModule prefork.c>
    ServerLimit            384
    MaxRequestWorkers      384
    StartServers           8
    MinSpareServers        8
    MaxSpareServers        16
    MaxConnectionsPerChild 4000
</IfModule>
# END APACHE_SMART_TUNER
------------------------------------------------
```

JSON output mirrors the same data structure for pipelines:

```json
{
  "version": "1.17.0",
  "mode": "analyze",
  "mpm": "prefork",
  "tier": "LOW-MID",
  "total_ram_mb": 4096,
  "cpu_cores": 4,
  "apache_running": true,
  "observed_processes": 180,
  "avg_httpd_mb": 12,
  "apache_budget_mb": 1434,
  "apache_budget_pct": 0.35,
  "apache_budget_source": "tier",
  "log_file": "/var/log/apache-smart-tuner.log",
  "recommended_block": "# BEGIN APACHE_SMART_TUNER\n# Apache Smart Tuner v1.17.0 (Tier: LOW-MID, MPM: prefork)\nTimeout 120\nKeepAlive On\nMaxKeepAliveRequests 100\nKeepAliveTimeout 5\n\n<IfModule prefork.c>\n    ServerLimit            384\n    MaxRequestWorkers      384\n    StartServers           8\n    MinSpareServers        8\n    MaxSpareServers        16\n    MaxConnectionsPerChild 4000\n</IfModule>\n# END APACHE_SMART_TUNER\n",
  "current": {
    "ServerLimit": "256",
    "MaxRequestWorkers": "256",
    "ThreadsPerChild": "",
    "StartServers": "5",
    "MinSpareThreads": "",
    "MaxSpareThreads": "",
    "MinSpareServers": "5",
    "MaxSpareServers": "10",
    "MaxConnectionsPerChild": "0"
  },
  "recommended": {
    "ServerLimit": "384",
    "MaxRequestWorkers": "384",
    "ThreadsPerChild": "",
    "StartServers": "8",
    "MinSpareThreads": "",
    "MaxSpareThreads": "",
    "MinSpareServers": "8",
    "MaxSpareServers": "16",
    "MaxConnectionsPerChild": "4000"
  }
}
```

## Safety details
- **Backups:** Target include files are copied with a `.bk-YYYYmmddHHMMSS` suffix before any write.
- **Scrub legacy blocks:** Existing `prefork.c`, `worker.c`, `event.c`, and `mpm_*` blocks inside the target include are removed before the new tuner block is written.
- **Validation:** Runs `apachectl`/`apache2ctl`/`httpd -t` before and after apply; failures trigger an automatic rollback.
- **Reload control:** When Apache is running, the tuner reloads automatically unless `--no-reload` is set; if Apache is stopped, it writes the config and logs the pending start.

## Versioning
The project follows semantic versioning. The current release is **1.17.0**.

## License
GPL-3.0-or-later. Authored by **Ryan MacDonald <ryan@rfxn.com>**.
