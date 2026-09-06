# Troubleshooting

Symptoms, causes, and fixes. If your problem is that a plugin destroyed something, go to [Backup and recovery](backup-and-recovery.md) instead.

## Nothing seems to happen when I click a button

**The job is running in the background.** The button re-enabling does **not** mean the job finished, and on a large catalogue a run can take 5 to 15 minutes or more.

- In Stream-Mapparr, watch **📊 View Check Progress**, then read **📋 View Last Results** when it is done.
- In the other plugins, use their status or results action.
- From the shell: `docker logs -f dispatcharr | grep "Stream-Mapparr"` and wait for `✅ COMPLETED`.

Do not queue another action while one is running. Only one long-running job runs at a time.

## The plugin says an operation is already running, but nothing is

A previous run crashed and left its lock behind. The lock clears itself after 10 minutes. To clear it now, use **🔓 Clear Operation Lock** (Stream-Mapparr) or the equivalent action on the plugin.

If scheduled runs stopped firing, use **🧹 Cleanup Orphaned Tasks** to remove stale scheduler entries.

## My scheduled run never fires

Check the time format. **Stream-Mapparr and Event Channel Managarr accept `HHMM` only** (`0400`), not `04:00`. A value containing a colon is silently ignored, with no error to tell you why.

Check the time zone too. Both plugins follow **Dispatcharr's global Time Zone** (Settings, then General). Neither has a Timezone setting of its own any more. See [Scheduling and run order](scheduling.md).

Note that **Channel Mapparr has no scheduler at all**, and that the only thing **EPG-Janitor** can schedule is its EPG Freshness Watchdog. Everything else in both plugins is triggered by hand.

In **IPTV Checker**, check that you clicked **💾 Save Schedule** after editing the times, and that two separate cron expressions are joined with a semicolon rather than a comma. That plugin has no on/off switch for scheduling and no timezone setting of its own: a non-empty **Scheduled Check Times** field is what arms it, and the times follow Dispatcharr's global Time Zone.

## The plugin refuses to run against my profile

Stream-Mapparr will not run against the default **"All"** profile, and several plugins require a profile name to be set. Create a real Channel Profile first, as described in [Prerequisites](prerequisites.md).

## My hidden event channels keep coming back

Dispatcharr's **Auto Channel Sync** re-enables every channel in a synced group on each M3U refresh, silently undoing Event Channel Managarr's work.

Turn on **Auto-rescan after M3U refresh** in Event Channel Managarr so it re-hides them immediately after each refresh.

## My channels keep un-hiding, and I use two plugins

Stream-Mapparr's **Manage Channel Visibility** and Event Channel Managarr both toggle channel visibility, using different criteria, and they will undo each other.

**Run Stream-Mapparr first, then Event Channel Managarr.** See [Scheduling and run order](scheduling.md).

## Auto-Match matched nothing against a brand-new EPG source

This is expected, and it is not a bug. Dispatcharr only imports program data for EPG entries that are already mapped to a channel, so a freshly added EPG source starts with **zero** programs. Every candidate is then rejected for having no program data.

The fix, in order:

1. Turn **Allow EPG Without Program Data** on.
2. Run Auto-Match to assign the EPG IDs.
3. Refresh the EPG source so Dispatcharr backfills the program data.
4. Turn **Allow EPG Without Program Data** back off.

Leaving it on permanently is what re-introduces "No Program Information Available".

## Too many channels are being skipped or unmatched

Lower the **Match Sensitivity**. It means the same thing in every plugin: the minimum similarity a name must reach. `Relaxed` is 70, `Normal` is 80, `Strict` is 90, `Exact` is 95.

If a specific channel simply will not match at any sensitivity, use that plugin's **Custom Aliases** setting to force it. Do not confuse aliases with **Ignore Tags**, which strips noise such as `[FHD]` from names before comparing them.

## I am getting false matches

Raise **Match Sensitivity** to `Strict`.

On a multi-country M3U, also turn on **Restrict Matching To Same Country** (Stream-Mapparr). Remember that matching reads the **stream name only** and never its group, so putting streams in tidy groups does not prevent a Spanish channel matching a UK one.

If an over-the-air channel is grabbing odd streams, note that callsigns which are also ordinary English words (KING, WHO, WOLF, WAVE, WOOD) are deliberately guarded and require corroboration, so `24/7 KING OF THE HILL` will not be attached to the Seattle NBC station.

## Dispatcharr starts returning errors during a long run

Raise **Rate Limiting** from `None` to `Low` or `Medium`. This paces the plugin's writes. It is the right response to 429 or 5xx errors during bulk operations.

## A stream probe interrupted somebody's viewing

Probing opens a real connection to your provider. If your provider caps concurrent connections, a probe can knock a viewer off.

Probe when nobody is watching, and be aware that IPTV Checker's full scan is slow enough (many hours on a large catalogue) that it should be windowed overnight.

## The whole web UI hangs after a plugin run

Update Stream-Mapparr. A long matching job could run inside the web request and block the server, which made the UI stop responding and eventually return a 504, even though streams kept playing. This is fixed in `1.26.1931038` and later.

## My provider names have invisible characters and nothing matches

Some providers pad stream names with invisible Unicode characters around a decorative glyph. These used to survive name normalization and destroy the match rate for everything from that provider.

Update to the current release of the plugin. Stream-Mapparr, Lineuparr, EPG-Janitor and Channel Mapparr all strip these characters now.

## Still stuck

Check the plugin's own repository for open issues, and the [Discord](https://discord.gg/Sp45V5BcxU) for discussion. Links to every plugin are in [Plugin Links](reference/plugin-links.md).
