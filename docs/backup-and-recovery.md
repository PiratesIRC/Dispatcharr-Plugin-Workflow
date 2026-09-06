# Backup and recovery

Every plugin in this guide changes many rows at once, and several of them delete things. None of them has an undo button. A backup is the undo button.

This page covers how to take one, what each plugin can actually destroy, and how to restore if a run goes wrong.

## Take a backup

Dispatcharr has a backup facility built in, and it is the one to use. It writes a full database dump to a zip file under `/data/backups/` inside the container.

Many installs already run one nightly, so check what you have before assuming you have nothing:

```bash
docker exec dispatcharr ls -lh /data/backups/
```

If you see recent `dispatcharr-backup-*.zip` files, a scheduled backup is running and you can simply confirm the newest one is recent enough. If not, take one before you run anything destructive.

!!! tip "Take a fresh one before your first apply"
    A nightly backup from last night is fine for a routine run. Before the first time you apply a *new* action, or before you enable any of the automatic triggers, take a fresh backup so the restore point is immediately before the change.

## What each plugin can destroy

Read this before your first run of anything. These are the behaviours that surprise people, and each is explained in detail on the plugin's own page.

| Plugin | What it can destroy |
| --- | --- |
| **[Stream-Mapparr](workflow/03-stream-mapparr.md)** | **Match & Assign replaces a channel's entire stream list.** It deletes the existing assignments and rebuilds from only what it matched in that run. Narrow M3U Sources and every stream from your other providers is dropped, failover backups included. **Manage Channel Visibility** disables every channel in the profile before re-enabling the ones with streams. |
| **[Lineuparr](lineuparr.md)** | **Full Sync and Apply Stream Match delete channels they could not match.** A lineup channel your M3U cannot supply is removed, not left empty. Full Sync is also not atomic, so a failure halfway leaves partial state. |
| **[EPG-Janitor](workflow/04-epg-janitor.md)** | **Strip Hidden EPG deletes program data**, not just the assignment, which can blank the guide for other channels that share the same EPG entry. The various Remove actions are permanent. |
| **[Channel Mapparr](workflow/02-channel-mapparr.md)** | Bulk renames every matched channel. **Tag Unknown Channels and both logo actions ignore Dry Run Mode** and write immediately. |
| **[Event Channel Managarr](workflow/05-event-channel-managarr.md)** | Bulk visibility changes, and it can strip EPG from channels as it hides them. |
| **[IPTV Checker](workflow/01-iptv-checker.md)** | Tags and can remove streams it judged dead. |

## Use Dry Run, but know its limits

Most plugins have a Dry Run or Preview mode that writes a CSV instead of touching the database. Use it every time before a first apply.

!!! warning "Dry Run does not cover everything"
    In **Channel Mapparr**, Dry Run Mode covers four actions: Rename Channels, Organize by Category, Import M3U Streams, and Create Channels From Streams. **Tag Unknown Channels, Apply Default Logo and Apply Per-Channel Logos write immediately even with Dry Run on.**

    Check each plugin's page rather than assuming the toggle protects you.

## Restore

Restoring is quick. It replays the dump back into the database without needing to rebuild the container, and it does not require a Postgres restart.

Restore from the Dispatcharr UI where the option is available. If you need to do it from the command line, the backup zips under `/data/backups/` are full database dumps and Dispatcharr ships the matching restore task, so a restore is a matter of pointing it at the file you want.

```bash
# see what you can restore to
docker exec dispatcharr ls /data/backups/
```

!!! danger "Never edit files under /data/db or /data/backups by hand"
    Do not run search-and-replace, `sed -i`, or any other byte-rewriting command across `/data`. The live PostgreSQL data directory lives there, and provider credentials are stored inside stream URLs, so a well-meaning "scrub my credentials from these files" command can match the database files themselves and corrupt them. If you need to change credentials, do it through the Dispatcharr UI.

## After a bad run

If a plugin run did something you did not expect:

1. **Stop.** Do not run another plugin to "fix" it. A second bulk operation on top of a bad one usually makes the restore harder, not easier.
2. Restore the most recent backup taken before the run.
3. Re-read the danger callout on that plugin's page, adjust the setting that caused it (very often it is **Overwrite Existing Streams** in Stream-Mapparr, or **Preserve Existing Streams** in Lineuparr), and re-run in Dry Run first.

If the run wiped stream assignments rather than channels, you do not necessarily need a full restore: re-running **Match & Assign** with the *correct*, unnarrowed M3U Sources selection will rebuild the stream lists from your full catalogue. Restoring is still the safer option if you are unsure.
