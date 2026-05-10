# Alternative: Lineuparr

**Repository:** [Dispatcharr-Lineuparr-Plugin](https://github.com/PiratesIRC/Dispatcharr-Lineuparr-Plugin)

## Goal

Mirror a real TV provider's channel lineup in one operation: create channel groups, create channels with the provider's channel numbering, fuzzy-match streams from M3U sources, assign EPG data, and assign logos. If you would rather not run the five-step workflow in this guide, Lineuparr is the one-click alternative.

!!! danger "Back up your database first"
    The full sync touches groups, channels, streams, EPG assignments, and logos in a single transaction. [Back up the Dispatcharr database](prerequisites.md#back-up-the-database) before running any actions.

## Plugin Flow

```mermaid
flowchart TD
    A["Pick Lineup File<br/>e.g. DIRECTV Premier"] --> B["Set M3U Source and<br/>Channel Profile"]
    B --> C["Validate Settings"]
    C --> D["Preview Stream Match<br/>CSV preview"]
    D --> E{"Review CSV"}
    E -->|adjust| F["Tune Match Sensitivity<br/>or Custom Aliases"]
    F --> D
    E -->|looks good| G["Full Sync<br/>groups, channels, streams, EPG, logos"]
    G --> H["Re-sort Streams by Quality"]
    H --> Z(["Mirrored provider lineup"])
```

## Supported Lineups

| Provider | Country | Approx. channels |
| --- | :---: | :---: |
| DIRECTV Premier | US | ~300 |
| DISH Top 250 | US | ~210 |
| Verizon FiOS | US | ~190 |
| Sky TV | UK | ~160 |
| Foxtel Platinum Plus | AU | ~140 |
| Telus Optik | CA | ~140 |
| ODIDO | NL | ~140 |

## Configuration Options

- **Lineup File:** Default `US_DirecTV-Premier_lineup.json`. Pick the JSON file matching the provider you want to mirror.
- **M3U Source:** The M3U source whose streams will be matched against the lineup's channels.
- **Channel Profile:** Where the mirrored channels should land.
- **Channel Group Prefix:** Optional prefix added to all groups Lineuparr creates (useful for keeping multiple lineups separate).
- **Category Detail:** Default `Normal`. Controls how granular the auto-created groups are.
- **Match Sensitivity:** Default `Normal`. Loosen if streams are not matching to known channels; tighten to reduce false positives.
- **Channel Numbering:** Default `Use Channel Database Numbers` — uses the provider's actual channel numbers.
- **Starting Channel Number:** Optional override if you want sequential numbers from a base value instead of provider numbers.
- **Order Matched Streams by Quality:** Default true. Sorts alternate streams by resolution / FPS.
- **Rate Limiting:** Default `None`. Raise if Dispatcharr returns errors during long runs.
- **Custom Channel Aliases (JSON):** Manual overrides for channels whose stream name does not match by fuzzy logic.
- **EPG Sources:** Default `All EPG sources`. Can be narrowed to specific sources.

!!! info "📸 Screenshot suggestion"
    **File:** `screenshots/lineuparr-settings.png`
    **Show:** the settings panel with **Lineup File**, **M3U Source**, and **Channel Profile** highlighted. These are the three required choices before any action will work.

## Action Sequence

1. Pick a **Lineup File**, **M3U Source**, and **Channel Profile**.
2. Run **Validate Settings** to confirm the lineup file loads, the M3U source is reachable, and the profile exists.
3. Run **Preview Stream Match** to see how M3U streams match against the lineup's channels. Review the CSV.
4. Run **Full Sync** to create groups, create channels, assign streams, assign EPG, and assign logos in one operation.
5. Alternatively, run the partial actions individually if you want finer control:
    - **Sync Channels Only** — create groups and channels without touching streams / EPG / logos.
    - **Apply Stream Match Only** — fuzzy-match streams to already-existing channels.
    - **Apply EPG Match** — assign EPG only.
    - **Assign Logos** — apply logos only.
6. Run **Re-sort Streams by Quality** to re-rank alternate streams attached to each channel.
7. Use **Clear CSV Exports** to clean up old preview files.

!!! info "📸 Screenshot suggestion"
    **File:** `screenshots/lineuparr-preview.png`
    **Show:** the Preview Stream Match CSV with `channel_number`, `channel_name`, `matched_stream`, and `match_score` columns. Lets readers see what a "good" match preview looks like before committing.

## Important Notes

- **Requires Dispatcharr v0.20.0+** and at least one M3U source.
- Lineuparr is an alternative to the [main workflow](index.md), not a replacement for the individual plugins. You can run Lineuparr first and still apply the per-plugin steps for fine-tuning afterwards (e.g., Stream-Mapparr's quality re-ranking, EPG Janitor's heal pass).
- Custom Channel Aliases is the escape hatch when fuzzy matching fails for a specific channel — define the override once and Full Sync will respect it on every run.

!!! info "📸 Screenshot suggestion"
    **File:** `screenshots/lineuparr-result.png`
    **Show:** the Dispatcharr Channels page after a successful Full Sync, with provider channel numbers and category groups visible. Demonstrates the end state.
