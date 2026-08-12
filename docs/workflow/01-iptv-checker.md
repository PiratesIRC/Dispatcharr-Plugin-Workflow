# 1. Stream Verification: IPTV Checker

**Repository:** [Dispatcharr-IPTV-Checker-Plugin](https://github.com/PiratesIRC/Dispatcharr-IPTV-Checker-Plugin) &nbsp; [![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/PiratesIRC/Dispatcharr-IPTV-Checker-Plugin) &nbsp; [![Discord](https://img.shields.io/badge/Discord-Discussion-5865F2?logo=discord&logoColor=white)](https://discord.gg/Sp45V5BcxU)

## Goal

Probe each stream with `ffprobe`, mark sources that are dead, low-framerate, or broken in a way that still answers the probe, and sync technical metadata (codecs, resolution, bitrate, FPS) back into the Dispatcharr database. The metadata gathered here is what Stream-Mapparr later uses to rank stream quality and filter out broken sources.

!!! danger "Back up your database first"
    This plugin makes bulk changes that cannot be undone. [Back up the Dispatcharr database](../prerequisites.md#back-up-the-database) before running any actions.

## Plugin Flow

```mermaid
flowchart TD
    A["Save settings"] --> B["Validate Settings"]
    B --> C["Load Groups"]
    C --> D["Start Stream Check"]
    D --> E{"Long run<br/>hours possible"}
    E --> F["View Check Progress"]
    E --> G["docker logs -f"]
    F --> H["View Last Results"]
    G --> H
    H --> I["Rename / Move /<br/>Delete dead channels"]
    H --> J["Add format suffix<br/>UHD/FHD/HD/SD"]
    H --> K["Export CSV"]
    I --> Z(["Metadata feeds Stream Mapparr"])
    J --> Z
    K --> Z
```

## Configuration Options

- **Groups to Check:** Comma-separated group names; empty = all groups. Wildcards supported (e.g., `US-*`, `*Sports*`).
- **Check Alternative Streams:** Probe every backup stream attached to a channel, not just the primary.
- **Connection Timeout** (default 10s), **Probe Timeout**, and **Dead Connection Retries** (default 3).
- **Per-Stream Cooldown** to avoid hammering a single host.
- **FFprobe Analysis Flags** and **FFprobe Analysis Duration** for tuning probe behavior on slow streams.
- **Streamlink-Only Hosts:** Comma-separated host patterns probed via `streamlink` instead of direct `ffprobe`.
- **Dead Channel Rename Format** and **Move Dead Channels to Group** (default `Graveyard`).
- **Low Framerate Rename Format** and **Move Low Framerate Group** (default `Slow`).
- **Video Format Suffixes:** Default `UHD, FHD, HD, SD, Unknown`.
- **Enable Parallel Checking** with **Number of Parallel Workers** for faster scans.
- **FFprobe Path:** Default `/usr/local/bin/ffprobe`.
- **Delete Dead Channels** with **Auto-Delete Confirmation** for fully removing dead channels rather than just renaming them.
- **Webhook URL** and **Send Webhook Notification** to fire an external notification when a check completes.
- **Enable Scheduled Checks**, **Scheduled Check Times** (cron syntax), **Scheduler Timezone**, **Export CSV for Schedule**.
- **Use Windowed Schedule** with **Window End Mode**, **Window Duration**, **Window End Time**, and **Reset Window Progress** for time-bounded scans that pick up where they left off.

### Finding streams that answer but are not watchable

A stream can pass `ffprobe` perfectly and still be useless: a black picture, a still frame, no sound, or a short file looping. Four optional checks catch those. **All four are off by default**, because the first three cost decode time on every stream.

- **Detect Blank-Screen Streams:** Decodes a few seconds with ffmpeg's `blackdetect` filter and marks all-black streams Dead under the `Black Screen` error type. Tuned by **Blank-Screen Sample** (default 6 seconds of video decoded), **Continuous Blank Required** (default 3 seconds, which should stay a few seconds below the sample so connection and keyframe latency do not eat the whole window), and **Blank-Screen ffmpeg Timeout** (default 20 seconds wall clock, after which the stream is left Alive rather than guessed at).
- **Detect Frozen-Video Streams:** Checks the same decoded sample for a continuous run of identical frames. **Frozen-Video Minimum** (default 4 seconds) is automatically reduced to fit inside the sample, because a value at or above the sample length could never be reached.
- **Detect Silent-Audio Streams:** Measures the mean volume of the decoded sample and marks anything at or below **Silence Threshold** (default -70 dBFS) as Dead. Streams with no audio track at all are skipped, since they cannot be silent. The default sits between digitally silent audio, which measures about -91 dB, and the quietest real channel measured at about -44 dB.
- **Detect Placeholder-File Streams:** Marks any stream reporting a fixed container duration as Dead under the `Placeholder File` error type. A live stream has no fixed duration, so this is a strong signal, and it **adds no probe time at all**. If you enable only one of these four, make it this one.

Blank-screen channels get their own rename format and destination group (**Blank-Screen Channel Rename Format**, **Move Blank-Screen Channels to Group**), and are deliberately excluded from the Dead rename and move actions so they are not tagged twice.

### Scheduled runs can act on their own results

Enabling **Scheduled Checks** only performs the scan. Each follow-up action has its own switch that decides whether the scheduled run also applies it, so an unattended scan can rename, move, delete, tag and email without you present:

- Renaming: **Rename Dead Channels**, **Rename Blank-Screen Channels**, **Rename Low Framerate Channels**, **Add Video Format Suffix**.
- Moving: **Move Dead Channels**, **Move Blank-Screen Channels**, **Move Low Framerate Channels**.
- Other: **Restore Recovered Channels**, which un-tags channels that have come back to life, and **Email Report After Scheduled Check**.

!!! danger "Delete Dead Channels After Scheduled Checks removes channels unattended"
    This one deletes rather than renames, on a schedule, with nobody watching. It is gated behind **Auto-Delete Confirmation**, which you must set to the literal word `DELETE` before it will do anything. Treat that gate as the safety feature it is, and make sure your database backups actually run before you arm it.

![IPTV Checker settings panel with parallel workers and scheduler visible](../screenshots/iptv-checker-settings.png)

## Action Sequence

1. Save scan preferences.
2. Run **Validate Settings** to confirm `ffprobe` path and configuration.
3. Run **Load Group(s)** to pull the channel list. Large lists load in the background.
4. Run **Start Stream Check**. The scan runs in a background thread and survives browser timeouts.
5. Monitor with **View Check Progress** for live ETA, then **View Last Results** or **View Results Table** when finished.
6. Optionally run **Rename Dead Channels**, **Move Dead Channels to Group**, **Rename Low Framerate Channels**, or **Add Video Format Suffix**. If you enabled blank-screen detection, **Rename Blank-Screen Channels** and **Move Blank-Screen Channels to Group** handle those separately.
7. Run **Restore Recovered Channels** to un-tag channels that were previously marked and now probe clean again. Without it, a channel that recovers keeps its dead-channel name.
8. Export with **Export Results to CSV**. Use **Clear CSV Exports** to clean up old exports. **Email Report** sends the results if you have mail configured.
9. Use **Cancel Stream Check** to stop a running scan, **Cleanup Orphaned Tasks** to clear stale Celery entries, or **Check Scheduler Status** to verify scheduled runs.

## Important Notes

!!! warning "Plan for hours, not minutes"
    A single-threaded check can take **24 hours on ~6,000 streams**. Even with parallel workers, large catalogs commonly run for several hours. Schedule the plugin rather than running it interactively.

### Recommended approach for large catalogs

- **Use the scheduler.** Enable **Scheduled Checks** with a cron time during your low-traffic hours rather than triggering scans manually. The scheduler also survives Dispatcharr restarts. If cron syntax is unfamiliar, build the expression interactively at [crontab.guru](https://crontab.guru/): for example, `0 3 * * *` runs once a day at 3 AM.
- **Tune parallel workers.** If your IPTV provider allows N concurrent connections, set **Number of Parallel Workers** to roughly **half of N**. Going higher risks the provider rate-limiting or banning your account; going much lower wastes time.
- **Consider Windowed Scheduling** for very large catalogs. The plugin will pick up where it left off on the next window, so a multi-day scan does not need to complete in one sitting.

### Watching progress

If the browser tab is open, **View Check Progress** gives a live ETA. If you closed the tab or the browser timed out, watch the container logs:

```bash
docker logs -f dispatcharr | grep "IPTV-Checker"
```

Wait for `✅ COMPLETED` in the log before queuing the next action. The UI button re-enabling does not always mean the scan finished.

### Other notes

- Metadata syncing happens automatically during the check and feeds Stream-Mapparr's quality ranking and dead-stream filtering.
- Run this plugin first so downstream plugins have accurate stream data to work with.
- The standard Dispatcharr container ships with `ffmpeg`, `ffprobe`, and `pytz` already installed, so no manual setup is needed.

![Enable Scheduled Checks with a cron expression and Windowed Schedule enabled](../screenshots/iptv-checker-scheduler.png)
