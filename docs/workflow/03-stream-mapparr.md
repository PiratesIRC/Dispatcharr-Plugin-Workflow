# 3. Stream Association — Stream Mapparr

**Repository:** [Stream-Mapparr](https://github.com/PiratesIRC/Stream-Mapparr)

## Goal

Match streams to channels via fuzzy logic, rank alternates by physical quality (resolution and FPS), prioritize preferred M3U sources, and toggle channel visibility based on whether streams are attached. Filters out dead 0x0 streams using IPTV Checker metadata and deduplicates stream names.

## Configuration Options

- **Dispatcharr URL / Admin Username / Admin Password.**
- **Profile Name:** Required. Must be a Channel Profile other than "All".
- **Channel Groups:** Comma-separated; empty = all.
- **Overwrite Existing Streams:** Replace current stream assignments (default true).
- **Fuzzy Match Threshold:** Default 85.
- **Visible Channel Limit:** How many duplicate channels to keep enabled per group (default 1).
- **Ignore Tags:** Tags stripped before matching (e.g., `[Dead], (Backup)`).
- **Rate Limiting:** None / Low / Medium / High. Raise it if you see 429 or 5xx errors.
- **Timezone** plus **Scheduled Run Times** (HHMM, comma-separated, e.g., `0400,1600`) for the built-in scheduler.
- **Channel Databases:** Toggle US, UK, CA, NL, etc. for OTA callsign matching.

## Action Sequence

1. Save credentials and select your Channel Profile (not "All").
2. Run **Preview Changes** to generate a CSV in `/data/exports/` showing proposed matches. The CSV header includes recommendations such as "lower threshold to 75" or "add 'UK' to ignore tags."
3. Review the CSV.
4. Run **Add Stream(s) to Channels** to apply.
5. For US over-the-air channels, run the dedicated callsign-matching action to refine OTA matches.
6. If upgrading from a pre-scheduler version, run **Cleanup Orphaned Tasks** to remove leftover Celery schedules.

## Important Notes

!!! danger "Background operations"
    The frontend shows "✅ Started in background" immediately, but progress and completion only appear in Docker logs. The button re-enabling does **not** mean the task finished.

- Watch progress with: `docker logs -f dispatcharr | grep "Stream-Mapparr"`. Wait for `✅ COMPLETED` before queuing the next action.
- Operations can take 5–15+ minutes on large catalogs.
- The Channel Profile must exist and must not be "All" — the plugin will refuse to run otherwise.
- To stop a runaway operation, restart the container: `docker restart dispatcharr`. The operation lock expires after 10 minutes on its own.
