# 4. EPG Maintenance — EPG Janitor

**Repository:** [Dispatcharr-EPG-Janitor-Plugin](https://github.com/PiratesIRC/Dispatcharr-EPG-Janitor-Plugin)

## Goal

Find channels that have an EPG source assigned but no actual program data — the "No Program Information Available" state in your TV guide — and either assign new EPG sources (Auto-Match) or replace broken assignments with working alternatives (Scan & Heal).

Both features use the same weighted scoring system: callsign 50 points, state 30, city 20, network 10, plus program data validation. This is EPG data-quality cleanup, not database size optimization.

## Configuration Options

- **Dispatcharr URL / Admin Username / Admin Password.**
- **Channel Profile Names:** Comma-separated; channels visible in any of the listed profiles are included.
- **EPG Sources to Match:** Comma-separated, ordered by priority (first = highest). Source names are validated and typos trigger warnings.
- **Hours to Check Ahead:** 1–168, default 12.
- **Channel Groups** *or* **Ignore Groups** — these are mutually exclusive.
- **EPG Name REGEX to Remove**, **Bad EPG Suffix** (default `[BadEPG]`), **Also Remove EPG When Adding Suffix**.
- **Heal: Fallback EPG Sources** and **Heal: Auto-Apply Confidence Threshold** (default 95).
- **Fuzzy matching toggles:** Ignore Quality Tags, Regional Tags, Geographic Prefixes, and Miscellaneous Tags (all on by default).

## Action Sequence

1. Save credentials and run **Validate Settings** to confirm API access, profile names, and group names.
2. Run **Preview Auto-Match (Dry Run)** to see what EPG would be assigned, with confidence scores. Review the CSV.
3. Run **Apply Auto-Match EPG Assignments** to assign validated EPG sources.
4. Run **Scan for Missing Program Data** to find any channels still showing "No Program Information Available."
5. Run **Scan & Heal (Dry Run)** to preview replacements for broken EPG. Confidence scores are in the CSV.
6. Run **Scan & Heal (Apply Changes)** to swap broken EPG for working alternatives at or above the threshold.
7. Use **Add Bad EPG Suffix to Channels**, **Remove EPG Assignments**, **Remove EPG by REGEX**, or **Remove ALL EPG from Group(s)** for manual cleanup.

## Important Notes

- **Auto-Match vs. Scan & Heal:** Auto-Match is for initial setup or bulk assignment. Scan & Heal is for ongoing repair when previously-working EPG breaks. Use Auto-Match first, then schedule periodic Scan & Heal runs for maintenance.
- Only EPG sources that actually have program data in the time window are assigned. Empty sources are skipped, which prevents the "No Program Information Available" problem from coming back through a bad assignment.
- Scan & Heal CSV status codes:
    - `HEALED` — applied
    - `REPLACEMENT_PREVIEW` — below threshold, needs manual review
    - `NO_REPLACEMENT_FOUND` — no working alternative in your configured sources
- EPG removal and renaming are permanent. All destructive actions require confirmation.
