# Bonus: Live Events — Event Channel Managarr

**Repository:** [Dispatcharr-Event-Channel-Managarr-Plugin](https://github.com/PiratesIRC/Dispatcharr-Event-Channel-Managarr-Plugin)

## Goal

Automatically toggle the visibility of existing event-style channels (PPV, sports, F1, fight nights) based on EPG data and channel name patterns. The plugin hides channels that currently have no event and shows ones that do, using a prioritized rule list.

!!! warning
    The plugin does not create new channels from keywords. The channels must already exist in your lineup, typically from event-style M3U entries.

## Configuration Options

- **Dispatcharr URL / Admin Username / Admin Password.**
- **Timezone** for scheduled runs.
- **Channel Profile Names** (required, comma-separated for multiple profiles).
- **Channel Groups** to narrow scope.
- **Name Source:** `Channel_Name` or `Stream_Name` for rule matching.
- **Hide Rules Priority:** Ordered, comma-separated list. Default:
  ```
  [InactiveRegex],[BlankName],[WrongDayOfWeek],[NoEventPattern],
  [EmptyPlaceholder],[PastDate:0],[FutureDate:2],
  [ShortDescription],[ShortChannelName]
  ```
  Other available rules: `[NoEPG]`, `[NumberOnly]`, `[PastDate:days:Xh]` (inline grace period override).
- **Regex: Channel Names to Ignore / Mark Channel as Inactive / Force Visible Channels.**
- **Duplicate Handling Strategy:** `lowest_number` / `highest_number` / `longest_name`, plus a **Keep Duplicate Channels** override.
- **Past Date Grace Period (Hours):** Default 4.
- **Auto-Remove EPG on Hide:** Default true.
- **Scheduled Run Times** (HHMM, comma-separated) and **Enable Scheduled CSV Export**.

## Action Sequence

1. Save credentials and the required Channel Profile name(s).
2. Adjust **Hide Rules Priority** if the default order does not fit your event channels.
3. Click **💾 Update Schedule** to save settings and activate any scheduled run times.
4. Run **🧪 Dry Run (Export to CSV)** and review the `reason` and `hide_rule` columns to confirm the right channels would be hidden or shown.
5. Run **🚀 Run Now** to apply visibility changes immediately.
6. Optionally run **🗑️ Remove EPG from Hidden Channels** to keep the guide clean.

## Important Notes

- Each scan covers all channels in the profile — visible and hidden — so a channel that picks up a new event will be re-shown automatically on the next run.
- The first matching rule in the priority list wins. Subsequent rules are not evaluated for that channel.
- The **Force Visible** regex always wins over hide rules. Useful for keeping news, weather, or always-on channels visible.
- Date extraction supports many formats (`Nov 8 16:00`, `12/25/2024`, `start:2024-12-25 20:00:00`, ISO formats, ordinal dates, and more). The grace period prevents premature hiding of events that run past midnight.
