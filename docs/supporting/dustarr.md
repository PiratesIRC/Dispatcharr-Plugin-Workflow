# Dustarr: find the channels nobody watches

**Repository:** [Dispatcharr-Dustarr-Plugin](https://github.com/PiratesIRC/Dispatcharr-Dustarr-Plugin) &nbsp; [![Discord](https://img.shields.io/badge/Discord-Discussion-5865F2?logo=discord&logoColor=white)](https://discord.gg/Sp45V5BcxU)

## Goal

Record which channels actually get watched, and report the ones that do not, so you can switch off the dead weight in a lineup the other plugins in this guide just spent their time cleaning up.

!!! success "This one changes nothing, so it needs no backup"
    Dustarr is read-only against Dispatcharr. It reads which channels have viewers and writes its own report files. It never renames a channel, never touches a stream, never assigns EPG and never toggles visibility. It is the only plugin documented here with no danger callout, and the only one you can install without thinking about a restore point first.

## Where it fits

Nowhere in the numbered workflow. It answers a different question from the rest of the guide: not "is this channel configured correctly" but "does anybody use it". Install it whenever you like and let it collect. The answer it produces is what tells you which channels are worth the attention of the other plugins, and which you could delete without anyone noticing.

## How it decides what counts as watched

- A collector samples Dispatcharr's live proxy state every 15 seconds and turns client counts into watch sessions. **Exactly one collector runs at a time** across all worker processes, elected by a lease, so nothing is counted twice.
- **A session becomes a watch only after two minutes.** Flipping through channels looking for something does not inflate the numbers. A shorter session is still recorded, as a *tune*, and that distinction is what produces the most useful list on the report.
- **A player may drop to zero clients for up to 90 seconds** during a reconnect without the session being treated as finished.

## The lists it produces

| List | What it means |
| --- | --- |
| **Never watched** | The dead weight, grouped by channel group so you can act on a whole group at once. |
| **Tuned but never qualified** | Channels you tried to watch and gave up on inside two minutes. **This is not a "barely used" list, it is almost certainly a broken list**: a dead source, a black picture, or a provider dropping the connection. Treat these as faults to investigate rather than channels to remove. |
| **Channels going cold** | Watched at some point, but not lately. Entries that are still being tuned are listed apart, because that shape also means broken rather than unwanted. |
| **Too new to judge**, **most used**, **least used** | The remainder. |

### What it refuses to judge

Roughly 70 percent of a normal lineup is held back from the unused lists on purpose, and that is the point rather than a limitation:

- **Pay-per-view and live event slots** idle between events, and an M3U sync renames the same channel row in place, so a slot that sat on "NO EVENT" for a month would otherwise read as permanently dead.
- **Local and over-the-air news** is the emergency tier. It is supposed to sit unused.
- **Sports** has a legitimate off season.
- **Channels the collector cannot see at all**, such as one on a stream profile set to Redirect, which writes none of the state the collector reads. These are reported separately as unobservable rather than counted as never watched, and that is decided from the profile's structure rather than from its name.

!!! warning "The rankings are not a summary of your viewing"
    An excluded channel is also absent from the Most used and Least used tables, even when it is watched every day. Those tables answer the question "what can I safely turn off", which is a narrower question than "what do we watch".

## Configuration Options

### How watching is measured

Changing any of these four restarts the collector, which forfeits any watch already in progress, so change them while nothing is streaming.

- **Poll interval (s):** Default 15. Must stay under Dispatcharr's 30 second metadata expiry, or live channels are missed between refreshes.
- **Minimum watch (s):** Default 120. Anything shorter is recorded as a tune rather than a watch.
- **Client gap grace (s):** Default 90. How long a player may drop to zero clients before the session is treated as over.
- **Session merge gap (s):** Default 120. A channel re-tuned inside this window continues the same watch.

### When a channel counts as unused

These are applied when the report is built, against the history already collected, so changing one **re-judges the past** rather than starting the record again.

- **Unused threshold (days):** Default 30. A channel younger than this cannot be called unused.
- **Cold threshold (days):** Default 30, with a floor of 7. A channel watched at some point but not inside this window is listed as going cold.
- **Never-watched alarm ceiling:** Default 0.98. Not a judgement about any channel. It decides when so much of the lineup looks dead that the collector is probably blind, at which point the report says it cannot be trusted rather than quietly claiming your household watches nothing.

### Channels that are never judged

- **Exclude auto-created channels:** Default on. This is what protects the event slots.
- **Excluded groups (comma separated)** and **Excluded name regex**, both shipped with sensible defaults covering the local, news and sports tiers.

### The report, and what happens to it

- **Rows in the Most used and Least used tables:** Default 20.
- **Send notifications to Newsflasharr:** Default off. Requires the [Newsflasharr](newsflasharr.md) plugin, which is what actually sends the mail.
- **Scheduled report:** Off, Daily, Weekly (the default, Monday) or Monthly, always at 03:00.
- **Delete saved reports older than (days):** Default **0, which keeps everything**. It removes only this plugin's own dated report files, never the live report, and always leaves at least one of each kind.

## Action Sequence

1. Install, enable, and **restart the Dispatcharr container**. The collector starts when the plugin is constructed in each worker, and a hot reload does not reliably re-run that.
2. Run **✅ Validate** first. It writes nothing, and it is the action that tells you whether the collector is alive, whether the schedule is firing, and whether email is ready.
3. Wait. See the note below about your first month.
4. Run **📊 Summary** for the tracking window, the coverage figure and the never-watched count.
5. Run **📈 Build report** to write the HTML report and the CSV now, and to email them if notifications are on.

!!! warning "A working Build report button does not prove the schedule works"
    The button runs in the web worker; the schedule runs on a Celery worker. They are different processes and only one of them is being tested when you press the button. **✅ Validate** reports the age of the last scheduled run, which is the signal that actually answers the question.

## Important Notes

!!! note "Your first month of reports will carry a red 'not trustworthy' banner, and that is the plugin working"
    The unused threshold defaults to 30 days, and a dataset younger than that cannot honestly call anything unused. There is no way to shorten this by importing history, because Dispatcharr does not keep the state Dustarr reads, so a fresh installation really does start at zero. Wait it out, or lower the threshold if you are comfortable judging on a shorter window.

- **Requires Dispatcharr v0.20.0+.**
- **It needs no internet access of any kind.** It never contacts your provider, never checks for its own updates, and fetches nothing while rendering a report.
- The report is a self-contained HTML page with sortable tables and inline charts, and no external stylesheet, script, font or image. It opens straight off disk and renders the same offline or as an email attachment.
- Newsflasharr is optional. With it absent or disabled, nothing is sent and nothing fails.
