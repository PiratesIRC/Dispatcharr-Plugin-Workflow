# 1. Stream Verification — IPTV Checker

**Repository:** [Dispatcharr-IPTV-Checker-Plugin](https://github.com/PiratesIRC/Dispatcharr-IPTV-Checker-Plugin)

## Goal

Probe each stream with `ffprobe`, mark dead or low-framerate sources, and sync technical metadata (codecs, resolution, bitrate, FPS) back into the Dispatcharr database. The metadata gathered here is what Stream-Mapparr later uses to rank stream quality and filter out broken sources.

## Configuration Options

- **Dispatcharr URL / Username / Password** for API authentication.
- **Groups to Check:** Comma-separated group names; empty = all groups.
- **Check Alternative Streams:** Probe every backup stream attached to a channel, not just the primary.
- **Connection Timeout** (default 10s) and **Dead Connection Retries** (default 3).
- **Dead Channel Rename Format** and **Move Dead Channels to Group** (default `Graveyard`).
- **Low Framerate Rename Format** and **Move Low Framerate Group** (default `Slow`).
- **Video Format Suffixes:** Default `4k, FHD, HD, SD, Unknown`.
- **Enable Parallel Checking** with **Number of Parallel Workers** for faster scans.
- **FFprobe Path:** Default `/usr/local/bin/ffprobe`.
- **Enable Scheduled Checks**, **Scheduled Check Times** (cron syntax), **Scheduler Timezone**, **Export CSV for Schedule**.

## Action Sequence

1. Save credentials and scan preferences.
2. Run **✅ Validate Settings** to confirm API connection and `ffprobe` path.
3. Run **📥 Load Group(s)** to pull the channel list. Large lists load in the background.
4. Run **▶️ Start Stream Check**. The scan runs in a background thread and survives browser timeouts.
5. Monitor with **📊 View Check Progress** for live ETA, then **📋 View Last Results** when finished.
6. Optionally run **✏️ Rename Dead Channels**, **⚰️ Move Dead Channels to Group**, **🐌 Rename Low Framerate Channels**, or **🎬 Add Video Format Suffix to Channels**.
7. Export with **💾 Export Results to CSV**.

## Important Notes

- Requires `ffmpeg` and `ffprobe` inside the Dispatcharr container. The scheduler also requires `pytz`.
- Metadata syncing happens automatically during the check and feeds Stream-Mapparr's quality ranking and dead-stream filtering.
- Run this plugin first so downstream plugins have accurate stream data to work with.
