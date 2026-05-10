# Screenshots

UI captures referenced from the guide pages. Drop image files here matching the filenames the pages already point to.

## Expected files

The guide currently calls for these screenshots (placeholders are present in the corresponding `.md` files as `📸 Screenshot suggestion` admonitions). Replace each admonition with a real image when the file is added.

### Home + prerequisites

- `dispatcharr-plugins-installed.png` — Plugins page with all six plugins installed
- `prerequisites-channel-profile.png` — Channel Profile drop-down + create dialog

### IPTV Checker

- `iptv-checker-settings.png` — full settings panel, parallel workers + scheduler visible
- `iptv-checker-results.png` — View Last Results table with mixed working / dead / slow streams
- `iptv-checker-scheduler.png` — Enable Scheduled Checks with cron + Use Windowed Schedule

### Channel Mapparr

- `channel-mapparr-settings.png` — settings with Channel Databases, Match Sensitivity, Dry Run Mode
- `channel-mapparr-dry-run-csv.png` — dry-run CSV excerpt with original / proposed / score columns
- `channel-mapparr-category-groups.png` — Channels page after Organize by Category

### Stream Mapparr

- `stream-mapparr-profile-selection.png` — Profile Name field with non-"All" profile selected
- `stream-mapparr-preview-csv.png` — dry-run CSV header + recommendation block
- `stream-mapparr-docker-logs.png` — `docker logs -f` running through to `✅ COMPLETED`

### EPG Janitor

- `epg-janitor-settings.png` — EPG Sources to Match + the two confidence thresholds
- `epg-janitor-auto-match-preview.png` — Preview Auto-Match CSV
- `epg-janitor-heal-preview.png` — Heal Preview CSV showing all three status codes
- `epg-janitor-before-after.png` — TV Guide before / after EPG repair

### Event Channel Managarr

- `event-channel-managarr-hide-rules.png` — Hide Rules Priority field + regex fields
- `event-channel-managarr-dry-run.png` — Dry Run CSV with `reason` and `hide_rule` columns
- `event-channel-managarr-before-after.png` — Channels page before / after a Run Now

### Lineuparr

- `lineuparr-settings.png` — Lineup File / M3U Source / Channel Profile fields
- `lineuparr-preview.png` — Preview Stream Match CSV
- `lineuparr-result.png` — Channels page after Full Sync

## Reference syntax

```markdown
![Validate Settings result](../../screenshots/iptv-checker-settings.png)
```

Path is relative from the doc page to this folder. From a page at `docs/workflow/01-iptv-checker.md`, that is `../../screenshots/...`. From `docs/index.md` or `docs/lineuparr.md`, it is `../screenshots/...`.

## Conventions

- PNG for UI screenshots; JPG only if the screenshot includes a photographic element.
- Keep images under ~500 KB where reasonable.
- Filenames use lowercase and hyphens: `<plugin-slug>-<screen>.png`.
