# 2. Channel Organization — Channel Mapparr

**Repository:** [Dispatcharr-Channel-Maparr-Plugin](https://github.com/PiratesIRC/Dispatcharr-Channel-Maparr-Plugin)

## Goal

Standardize the names of channels that already exist in Dispatcharr using country-specific channel databases (US, UK, CA, AU, BR, DE, ES, FR, IN, MX), and optionally re-organize them into category groups (News, Sports, Entertainment, etc.).

!!! warning
    This plugin does not create channels from streams. It works only on channels already in your lineup.

## Configuration Options

- **Dispatcharr URL / Admin Username / Admin Password.**
- **Channel Databases:** Comma-separated 2-letter country codes (default `US`).
- **Fuzzy Match Threshold:** 0–100 similarity score (default 85). Higher = stricter.
- **Channel Groups to Process** and **Channel Groups for Category Organization** to limit scope.
- **OTA Channel Name Format:** Template using `{NETWORK}`, `{STATE}`, `{CITY}`, `{CALLSIGN}` (default `{NETWORK} - {STATE} {CITY} ({CALLSIGN})`).
- **Ignored Tags:** Comma-separated tags stripped before matching (handles `[]` and `()`).
- **Suffix for Unknown Channels:** Default `[Unk]`.
- **Default Logo:** Logo display name from Dispatcharr's Logo Manager.

## Action Sequence

1. Save settings and select the country databases you want loaded.
2. Run **Load/Process Channels** to load and match channels against the selected databases.
3. Run **Preview Changes (Dry Run)** to export a CSV of proposed renames. Review before committing.
4. Run **Rename Channels** to apply standardized names.
5. Run **Add Suffix to Unknown Channels** to flag whatever did not match.
6. Optionally run **Apply Default Logos** for channels missing artwork.
7. For category sorting, run **Category Groups Dry Run**, then **Organize Channels by Category** to move channels into matching category groups (creates new groups when needed).

## Important Notes

- Use the *display name* from the Dispatcharr Logos page for **Default Logo**, not the filename.
- If channels are skipped because of minor spelling differences, lower the **Fuzzy Match Threshold** to 75–80.
- API tokens are cached for 30 minutes to reduce auth overhead.
