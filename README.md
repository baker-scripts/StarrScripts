# StarrScripts

[![ShellCheck](https://github.com/baker-scripts/StarrScripts/actions/workflows/shellcheck.yml/badge.svg)](https://github.com/baker-scripts/StarrScripts/actions/workflows/shellcheck.yml)
[![Pre-commit](https://github.com/baker-scripts/StarrScripts/actions/workflows/pre-commit.yml/badge.svg)](https://github.com/baker-scripts/StarrScripts/actions/workflows/pre-commit.yml)
[![Release](https://img.shields.io/github/v/release/baker-scripts/StarrScripts)](https://github.com/baker-scripts/StarrScripts/releases)

Bash and Python scripts for Starr apps (Radarr, Sonarr) and the tools around them: qui cross-seeding, qBittorrent automation, app updaters, duplicate finders, and ZFS snapshot cleanup. Created by [Bakerboy448](https://github.com/bakerboy448/) unless noted.

Many scripts hardcode paths, service names, and hosts in variables at the top. Read a script and edit those before running it. Several use `sudo`, delete files, or restart services.

Do not edit `.sh` files on Windows without keeping `LF` line endings; `CRLF` breaks Bash scripts.

## Scripts

| Script | Purpose |
| --- | --- |
| `qui-xseed.sh` | Radarr/Sonarr custom script: triggers a [qui](https://github.com/autobrr/qui) data-based cross-seed search after an import |
| `qbm-api-trigger.sh` | Runs [qbit_manage](https://github.com/StuffAnThings/qbit_manage) commands through its Web API for one torrent hash |
| `qbm-qbit.sh` | Waits 10 minutes, then runs qbit_manage once, under a lock file |
| `qbm-update.sh` | Updates a qbit_manage git checkout and restarts its service |
| `pmm-update.sh` | Updates a [Plex Meta Manager](https://github.com/meisnate12/Plex-Meta-Manager) checkout and restarts its service |
| `pic-update.sh` | Updates a [Plex-Image-Cleanup](https://github.com/meisnate12/Plex-Image-Cleanup) checkout and restarts its service |
| `omegabrr_upgrade.sh` | Installs the latest [Omegabrr](https://github.com/autobrr/omegabrr) `linux_x86_64` release and restarts its service |
| `notifiarr-branch-builder.sh` | Builds the Notifiarr client from a chosen branch, replaces the installed binary, and restarts the service |
| `dupe.sh` | Runs `jdupes` over two directories and hardlinks duplicates |
| `radarr_dupefinder.sh` | Prints directories containing more than one video file |
| `sonarr_dupefinder.sh` | Prints directories containing the same episode number more than once |
| `merge_folders.py` | Merges several directories into one without overwriting existing files |
| `zfsburn.sh` | Destroys ZFS autosnapshots beyond per-type count limits |
| `backup_config.sh` | Archives `/.config` (minus media) and rsyncs the archive to a backup path |
| `f2b-dump.sh` | Collects Fail2ban status and config files and uploads them to termbin.com |
| `servarr/servarr_bot_merge.sh` | Rebases the Servarr Wiki bot branch onto `master` and pushes it |

## Quick start: qui cross-seed trigger

1. Copy `.env.sample` to `.env` next to `qui-xseed.sh` and fill in the `QUI_*`, `TORRENT_CLIENTS`, and `USENET_CLIENTS` values.
2. Docker users: mount `.env` and `qui-xseed.sh` into the Starr app's `/config`.
3. Radarr: `Settings` -> `Connect` -> `Custom Script`, enable "On Import" and "On File Upgrade".
4. Sonarr: `Settings` -> `Connect` -> `Custom Script`, enable "On Import Complete".
5. Save and use Test.

On each import the script adds the parent directory to qui as a cross-seed data directory, triggers a scan, polls qui every 10, 20, ... seconds (up to a day) until it finishes, then removes the directory. For Sonarr episodes it scans the whole season folder. Download IDs already in `LOGID_FILE` are skipped. Only warnings and errors go to stderr, so the Starr app does not raise notifications for routine output. It replaces the legacy `xseed.sh` removed in v4.0.0.

## Configuration

Settings come from `.env` in the script's directory (see `.env.sample`) or the environment. Defaults below are the script defaults, which differ from some `.env.sample` examples.

### qui-xseed.sh

| Variable | Default | Purpose |
| --- | --- | --- |
| `QUI_HOST` | `localhost` | qui host |
| `QUI_PORT` | `7476` | qui API port |
| `QUI_APIKEY` | placeholder | qui API key |
| `QUI_TARGET_INSTANCE_ID` | `1` | qui instance ID |
| `QUI_QBIT_PATH_PREFIX` | empty | Path prefix as qBittorrent sees it, if it differs from the Starr app's |
| `QUI_TAGS` | empty | Comma-separated qBittorrent tags |
| `TORRENT_CLIENTS` | `Qbit` | Comma-separated Starr download client names that are torrent clients |
| `USENET_CLIENTS` | `SABnzbd` | Comma-separated usenet client names |
| `LOG_FILE` | `qui_xseed.log` in the script directory | Log file |
| `LOGID_FILE` | `qui_xseed_id.log` in the script directory | Processed download IDs |

### dupe.sh

| Variable | Default |
| --- | --- |
| `JDUPES_SOURCE_DIR` | `/mnt/data/media/` |
| `JDUPES_DESTINATION_DIR` | `/mnt/data/torrents/` |
| `JDUPES_OUTPUT_LOG` | `/mnt/data/jdupes.log` |
| `JDUPES_HASH_DB` | `/.config/jdupes_hashdb` |
| `JDUPES_COMMAND` | `/usr/local/bin/jdupes` |
| `JDUPES_EXCLUDE_DIRS` | `-X nostr:.RecycleBin -X nostr:.trash` |
| `JDUPES_INCLUDE_EXT` | `mp4,mkv,avi` |
| `DEBUG` | `false` |

### zfsburn.sh

| Variable | Default | Purpose |
| --- | --- | --- |
| `VERBOSE` | `1` | Per-snapshot logging |
| `MAX_FREQ`, `MAX_HOURLY`, `MAX_DAILY`, `MAX_WEEKLY`, `MAX_MONTHLY` | 4, 2, 7, 4, 3 | Count limit per snapshot type; `0` destroys all of that type |

### qbm-qbit.sh

| Variable | Default |
| --- | --- |
| `QBIT_MANAGE_LOCK_FILE_PATH` | `/var/lock/qbm-qbit.lock` |
| `QBIT_MANAGE_PATH` | `/opt/qbit-manage` |
| `QBIT_MANAGE_VENV_PATH` | `/opt/qbit-manage/.venv` |
| `QBIT_MANAGE_CONFIG_PATH` | `/opt/qbit-manage/config.yml` |
| `QBIT_MANAGE_OPTIONS` | `-cs -re -cu -tu -ru -sl -r` |

It reads `.env` from the current directory and needs the `lockfile` command (procmail package).

## Usage

```bash
# qBittorrent "Run external program on torrent completion" (Options -> Downloads)
/path/to/qbm-api-trigger.sh %I

# Updaters (run as the checkout's owner; pass "true" to force an update)
./qbm-update.sh [true]
./pmm-update.sh [true]
./pic-update.sh [true]

# Notifiarr from source (prompts interactively)
./notifiarr-branch-builder.sh [--repo-url URL] [--repo-dir DIR] [--bin-path PATH] [--branch BRANCH] [--reinstall-apt]

# Duplicates and cleanup
./dupe.sh
./radarr_dupefinder.sh [directory]
./sonarr_dupefinder.sh [directory]
./zfsburn.sh <dataset>
```

Script notes:

- `qbm-api-trigger.sh` posts `tag_update`, `share_limits`, `rem_unregistered`, and `recheck` for the hash to `http://127.0.0.1:4269/api/run-command` and logs to `run_qbit_manage_commands.log`. Edit `API_URL` and `COMMANDS` in the script. qbit_manage's Web API must be enabled and reachable.
- `qbm-update.sh`, `pmm-update.sh`, and `pic-update.sh` hard-reset the checkout to its upstream branch, so local changes there are lost. Default paths are `/opt/qbit_manage`, `/opt/Plex-Meta-Manager`, and `/opt/Plex-Image-Cleanup`; services restarted are `qbmanage`, `pmm`, and `pmm-image`.
- `omegabrr_upgrade.sh` restarts the `omegabrr@bakerboy448` service; change `service_name` first.
- `merge_folders.py` ships with its example call commented out. Set `source_dirs` and `target_dir`, uncomment the `atomic_moves(...)` call at the bottom, then run `python3 merge_folders.py`.
- `backup_config.sh` uses `/.config` as the source and `/mnt/backup-server/.config` as the destination; edit the variables at the top.
- `f2b-dump.sh` uploads its report to termbin.com, a public paste service. Review the output first or remove the upload line.
- `servarr/servarr_bot_merge.sh` expects a clone at `REPO_DIR` (set at the top of the script) with push access to `Servarr/Wiki`.

## Contributing

Issues and pull requests welcome. See the [contributing guidelines](https://github.com/baker-scripts/.github/blob/main/CONTRIBUTING.md) and [open an issue](https://github.com/baker-scripts/StarrScripts/issues) to discuss larger changes. Run `pre-commit run --all-files` before committing.

## Disclaimer

Provided as-is with no warranty. These scripts can delete files, change configuration, and call external APIs. Review them and test outside production first.

## License

[MIT](LICENSE)
