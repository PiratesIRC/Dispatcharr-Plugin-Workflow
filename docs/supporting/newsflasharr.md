# Newsflasharr: the plugin that sends everything else's mail

**Repository:** [Dispatcharr-Newsflasharr-Plugin](https://github.com/PiratesIRC/Dispatcharr-Newsflasharr-Plugin) &nbsp; [![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/PiratesIRC/Dispatcharr-Newsflasharr-Plugin) &nbsp; [![Discord](https://img.shields.io/badge/Discord-Discussion-5865F2?logo=discord&logoColor=white)](https://discord.gg/Sp45V5BcxU)

## Goal

Own all notification delivery for the suite in one place. Other plugins drop a small JSON event into a spool file and are finished; Newsflasharr routes it to any of seven destinations, with repeat collapsing, an hourly cap, quiet hours and retry configured once here instead of separately in every plugin.

!!! success "This one changes nothing in your lineup"
    Newsflasharr is read-only on Dispatcharr. It writes nothing outside its own `/data/newsflasharr/` directory, and it never creates or edits a channel, a stream or an Output Profile. No backup callout is needed before installing it.

## Why this page exists

Several pages in this guide say some version of "hands the report to the separate Newsflasharr plugin, which is what actually sends it". This is that plugin. **Five of the plugins documented here send through it**: IPTV Checker, Channel Mapparr, Stream-Mapparr, Lineuparr and [Dustarr](dustarr.md). EPG-Janitor and Event Channel Managarr do not.

It is optional in every one of those. With Newsflasharr absent or disabled, the calling plugin sends nothing and fails at nothing.

## The seven destinations

| Channel name | Goes to |
| --- | --- |
| `discord` | A Discord channel webhook. |
| `webhook` | Any endpoint that accepts a JSON POST. |
| `ntfy` | An ntfy server and topic, yours or the public one. |
| `apprise` | An Apprise API endpoint, which fans out to its own long list of services. |
| `smtp` | Email, through a mail server you configure here. |
| `connect` | A webhook Integration you already set up on Dispatcharr's Connect page, reusing its URL and headers. |
| `ticker` | A banner drawn over live video. See the warning below. |

!!! warning "On a public ntfy server, the topic name is the password"
    Anyone who guesses the topic reads every notification this installation sends. Invent a long unguessable one. Never `alerts`, and never your hostname.

## Configuration Options

### Getting anything at all delivered

Three things have to line up, and missing the third is the usual reason a correctly configured channel delivers nothing.

1. Fill in **one** channel. ntfy needs both server and topic; email needs server, From address and Recipients.
2. Type that channel's name into **Default channels**: one or more of `discord`, `webhook`, `ntfy`, `apprise`, `smtp`, `connect`, `ticker`. **Blank means nothing is delivered.**
3. Click **✅ Validate**, then **🔔 Test**. Only a real send proves a channel works.

!!! note "Alerts are not lost while no channel is configured"
    With nothing enabled, alerts wait in the queue rather than being discarded.

### Routing

- **Default channels:** Where alerts go unless a rule says otherwise.
- **Routing rules:** A JSON list matching on source, event or severity, for example `[{"match": {"severity": "critical"}, "channels": ["smtp"]}]`. **A matched rule adds its channels to the defaults**, so adding a rule never silently un-routes anything, unless the rule sets `"exclusive": true`. An empty list means everything uses the defaults.
- **Critical alerts also go to:** A durable escalation without editing the rules JSON. It applies to every critical, including one an exclusive rule sent elsewhere. Warnings and information are unaffected.

### Noise control

- **Repeat window (minutes):** Default 10. The same alert repeating inside the window becomes one message now plus a summary when the window closes. **An alert that gets worse breaks the window and is sent at once**, so a warning can never swallow the critical that follows it. Setting this to 0 turns repeat collapsing off.
- **Critical repeat window (minutes):** Blank uses the window above. 0 means a critical repeat always comes through.
- **Max sends per hour:** Default 20. Anything over the limit waits and arrives as one summary. The default is chosen against Gmail relay caps: 20 an hour is 480 a day at worst.
- **Quiet hours:** `HH:MM-HH:MM` in Dispatcharr's own timezone. Blank is off. What waited arrives as one summary when the window ends.
- **Hourly cap bypass** and **Quiet hours bypass:** Both default to critical only. Alerts that bypass the hourly cap do not count towards it either, so a storm of criticals cannot spend the budget the rest of your alerts share.

### Privacy

**Your provider hostnames are stripped from outgoing messages**, worked out from the EPG sources and M3U accounts Dispatcharr already holds. **Additional hostnames to redact** covers anything it cannot infer, and **📋 Redaction list** prints exactly what is being removed.

!!! danger "An attached report file is sent verbatim and is not redacted"
    Redaction covers message text. A file attached to an email goes as it is. Check what a report contains before routing it off this machine.

### Dispatcharr's own events

Separately from the other plugins, Newsflasharr can subscribe to events Dispatcharr itself publishes: blocked M3U and EPG requests, failed logins, channel errors and failovers, and recordings. Each has its own switch, all off unless you turn them on.

### The on-screen ticker

Draws alert text over live video. Two conditions that are easy to miss:

- It needs a Dispatcharr **Output Profile carrying the drawtext filter**. The **🎬 Ticker filter** action prints the exact ffmpeg parameters built from your appearance settings, for you to paste into that profile.
- It reaches **only devices whose playlist URL carries `?output_profile=<id>`**. A client on the default profile sees nothing, however the ticker is configured.

Adding `ticker` to Default channels does not put every alert on your television: it still obeys the ticker's own minimum severity, which defaults to critical only.

### Advanced

- **Queue check interval (s):** Default 5. This is the delay before a new alert is picked up, so it is the setting to look at when notifications feel *late* rather than *missing*.
- **Max event age (hours):** Default 24. An undelivered event older than this is dropped as expired. **That includes criticals**, and nothing is retried afterwards.
- **Expect a report every N days:** Blank is off. Status turns red once a report that *was* arriving stops. It cannot flag one that never arrived at all, so confirm delivery works once before relying on it.

## Action Sequence

1. Install the plugin. The archive contains a single folder named `newsflasharr`, which is the name Dispatcharr keys the plugin on, so do not rename it.
2. **Restart the Dispatcharr container. This one is not optional.** The worker that collects and delivers events starts when the plugin is constructed in each web worker, and a hot reload does not reliably re-run that. Without a restart the plugin loads and looks healthy while nothing is delivered.
3. Enable the plugin, then reload the page before changing anything. An old tab silently reverts your settings when it saves.
4. Fill in at least one channel, and name it in **Default channels**.
5. Click **✅ Validate**. Saving the form on its own arms nothing, because Dispatcharr gives plugins no hook that runs after a save.
6. Click **🔔 Test** for each channel you configured, naming one channel at a time.
7. Use **🧭 Show routing** to print where each kind of notification actually ends up, and **📊 Status** for the heartbeat, the queue depth and anything stuck.

## Important Notes

- **A calling plugin makes one call and is finished.** The call writes a small file and returns. It never blocks and never raises, so a plugin that reports a queued notification is telling you the event reached the spool, not that it reached your inbox.
- **Delivery is at-least-once per channel.** A duplicate is possible after a crash. That is a deliberate trade-off in favour of not losing an alert.
- A notification carries one attachment. A plugin set to email both an HTML and a CSV report therefore sends two separate messages rather than one message with two files.
