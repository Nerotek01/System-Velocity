<p align="center">
  <img src="https://img.shields.io/badge/version-2.2.3-blue?style=for-the-badge" alt="Version 2.2.3">
  <img src="https://img.shields.io/badge/platform-velocity%203.3.0%2B-informational?style=for-the-badge" alt="Velocity 3.3.0+">
  <img src="https://img.shields.io/badge/java-21-orange?style=for-the-badge" alt="Java 21">
  <img src="https://img.shields.io/badge/license-proprietary-red?style=for-the-badge" alt="Proprietary license">
</p>

<h1 align="center">System Velocity</h1>

<p align="center"><strong>Proxy-layer security &amp; maintenance for Velocity 3.3.0+ — standalone, zero dependencies</strong></p>

System Velocity is a security and maintenance plugin that runs entirely on the Velocity proxy. A single chat pipeline blocks null bytes, flooding, oversized messages, Log4Shell (CVE-2021-44228) payloads and malicious Unicode before backend servers ever see them, while a timezone-aware scheduler restarts the proxy daily and an automated cleaner keeps the proxy's `logs/` directory within retention and size limits. It is fully standalone: no backend plugin, no database, no external service.

> The "System" name is product-line branding — the plugin is inspired by the architecture of Hypixel's System-Proxy (inspiration only, no affiliation) and is not coupled to any backend plugin of the same family.

## Highlights

- **Exploit defense at the proxy layer** — the chat pipeline (null-byte → flood → length → Log4Shell → Unicode) denies the first matching payload before it reaches backend servers.
- **Deep Log4Shell detection** — case-insensitive `jndi:` matching, nested `${${lower:j}ndi:...}` brace flattening up to depth 4, dangerous-lookup patterns, payload-scheme detection (ldap, ldaps, rmi, dns, iiop, nds, nis, corba), a >4096-character anti-truncation rule, and sanitized console output so blocked payloads cannot forge logs.
- **Unicode filtering with locale pass-through** — zero-width characters, RTL/bidi overrides and isolates, control characters and invisible look-alikes are removed with full surrogate-pair awareness, while normal Persian and Arabic text, standard punctuation and emoji pass through untouched.
- **Violation tracking with configurable kicks** — per-player, per-type counters; Log4Shell kicks after 3 violations (default on), Unicode after 5 (default off); optional broadcast alerts to staff.
- **Timezone-aware daily restart** — fixed HH:mm in a configurable IANA timezone (default 04:00, Asia/Tehran), with countdown warnings that survive configuration reloads.
- **Symlink-safe log cleaner** — retention- and size-based cleanup for the proxy's own `logs/` directory; never follows or deletes symbolic links.
- **Self-verifying** — `/systemproxy test` runs built-in Log4Shell and Unicode test cases and prints per-case PASS/FAIL results with a summary.
- **Zero dependencies** — 10 classes, no plugin-messaging channels, no database, no external services; the only shaded library (snakeyaml) is relocated inside the jar to avoid classpath conflicts.

## Features

| Feature | Description | Default |
|---|---|---|
| Anti-Log4Shell filter | Multi-pass analysis of CVE-2021-44228 chat payloads: `${`-syntax stripping, nested-brace flattening (depth 4), dangerous-lookup patterns, payload schemes, >4096-char anti-truncation block, sanitized console logging | Enabled, kick after 3 violations |
| Anti-Unicode filter | Blocks zero-width, RTL/bidi and control characters plus suspicious invisibles; each malicious code point is replaced with `?` | Enabled, kick off (after 5 violations) |
| Anti-flood filter | Sliding window admits `max-messages-per-second` × `window-seconds` messages per player; mute cooldown once breached; primitive ring buffer with zero per-message allocation | Enabled — 5 msg/s over a 3 s window, 10 s cooldown |
| Length filter | Denies messages longer than `length-filter.max-length` | Enabled — 256 characters |
| Null-byte filter | Runs first in the pipeline; denies any message containing `\0` | Enabled |
| Violation tracker | Per-player totals and per-type breakdown (Log4Shell/Unicode); cleared on quit, pruned by a 5-minute sweep when that is disabled, cleared on kick and on proxy shutdown | Auto-clear on quit: on |
| Broadcast alerts | Staff holding `systemproxy.alerts` receive a configurable `{player}`/`{type}` alert; placeholders are interpolated after legacy deserialization so crafted usernames cannot inject color codes | Off (`alerts.broadcast: false`) |
| Daily restart scheduler | Restarts the whole proxy at a fixed time in a configurable timezone; countdown warnings at a configurable interval; countdown preserved across `/systemproxy reload`; all tasks cancelled on shutdown | Enabled — 04:00 Asia/Tehran, 60 s countdown every 15 s |
| Log cleaner | Manages the proxy's relative `logs/` directory: deletes entries older than `retention-days` and, when the size budget is exceeded, the oldest files beyond `keep-newest`; runs async at startup and on an interval; schedule refreshed on reload | Enabled — 7 days / 500 MB / keep newest 5, check every 30 min |
| Admin command | `/systemproxy` with aliases `/sp` and `/sysproxy`; tab completion gated by permission | `systemproxy.admin` |
| Built-in self-tests | 6 Log4Shell cases (including nested-obfuscation variants) and 4 Unicode cases with PASS/FAIL output | On demand via `/systemproxy test` |
| Config hardening | Defaults copied to disk on first run; malformed YAML falls back to built-in defaults; reload is atomic (both files parsed before swap); every value is typed, fallback-backed and clamped at the usage site | — |

## Commands

| Command | Permission | Description |
|---|---|---|
| `/systemproxy` (`/sp`, `/sysproxy`) | `systemproxy.admin` | Root admin command; no arguments prints help |
| `/systemproxy reload` | `systemproxy.admin` | Atomically reloads `config.yml` and `messages.yml`, reschedules the daily restart and the log cleaner; an active restart countdown is preserved, never cancelled |
| `/systemproxy status` | `systemproxy.admin` | Shows the enabled/disabled state of all five filters, the max message length, the restart time and timezone, and the log cleaner state |
| `/systemproxy test` | `systemproxy.admin` | Runs the built-in self-tests: 6 Log4Shell and 4 Unicode cases with PASS/FAIL lines and a `{pass}`/`{fail}` summary |
| `/systemproxy help` | `systemproxy.admin` | Prints the help message |
| *(receive alerts)* | `systemproxy.alerts` | Not a command: players with this permission receive the broadcast alert when `alerts.broadcast` is `true` |

Both permissions default to console-only unless granted through a permissions plugin.

## Quick start

1. Purchase a license at [store.hypeland.org](https://store.hypeland.org/plugins/system-velocity).
2. Complete Discord verification — staff confirm the order personally and deliver the plugin jar to you 1:1.
3. Drop the jar into the proxy's `plugins/` directory.
4. Restart the proxy — `config.yml` and `messages.yml` are generated in `plugins/systemproxy/` on first start.
5. Optionally run `/systemproxy status` and `/systemproxy test` on the console to confirm the filters and schedule.

Requirements: a Velocity 3.3.0+ proxy running Java 21 or newer. Nothing else is required — the plugin installs no channels, no databases and no services, and it does not need to be (and cannot be) connected to any backend plugin.

## Configuration

`plugins/systemproxy/config.yml` is generated on first start; every key has a safe default and values are clamped at the usage site.

### Chat pipeline thresholds

| Key | Default | Description |
|---|---|---|
| `flood-filter.enabled` | `true` | Enable the flood filter |
| `flood-filter.max-messages-per-second` | `5` | Rate basis; a window admits this value × `window-seconds` messages |
| `flood-filter.window-seconds` | `3` | Sliding-window length in seconds |
| `flood-filter.cooldown-seconds` | `10` | Mute cooldown once flooding is detected |
| `flood-filter.notify-player` | `true` | Tell the player the message was flood-blocked |
| `flood-filter.log-blocked` | `true` | Log flood blocks |
| `length-filter.enabled` | `true` | Enable the max-length filter |
| `length-filter.max-length` | `256` | Maximum chat message length in characters |
| `length-filter.notify-player` | `true` | Tell the player the message was too long |
| `length-filter.log-blocked` | `true` | Log over-length blocks |
| `null-byte-filter.enabled` | `true` | Deny messages containing a null byte (`\0`) |

### Exploit filters, kicks and alerts

| Key | Default | Description |
|---|---|---|
| `log4shell.enabled` | `true` | Enable the Log4Shell chat filter |
| `log4shell.notify-player` | `true` | Tell the player the message was blocked |
| `log4shell.log-blocked` | `true` | Log blocked attempts (sanitized payload) |
| `log4shell.kick-enabled` | `true` | Kick after repeated exploit attempts |
| `log4shell.max-violations` | `3` | Violations before kick |
| `unicode-filter.enabled` | `true` | Enable the Unicode injection filter |
| `unicode-filter.notify-player` | `true` | Tell the player the message was blocked |
| `unicode-filter.log-blocked` | `true` | Log blocked attempts (sanitized payload) |
| `unicode-filter.kick-enabled` | `false` | Kick after repeated malicious messages (off by default) |
| `unicode-filter.max-violations` | `5` | Violations before kick |
| `alerts.broadcast` | `false` | Broadcast violations to `systemproxy.alerts` holders |
| `alerts.auto-clear-violations-on-quit` | `true` | Clear violation data on disconnect; if `false`, a 5-minute sweep prunes offline players instead |

### Daily restart schedule

| Key | Default | Description |
|---|---|---|
| `restart.enabled` | `true` | Enable the daily proxy restart |
| `restart.time` | `"04:00"` | Restart time in HH:mm within `restart.timezone`; invalid values fall back to 04:00 with a warning |
| `restart.timezone` | `"Asia/Tehran"` | IANA ZoneId; invalid values fall back to UTC with a warning |
| `restart.warning-seconds-before` | `60` | Countdown length in seconds; `0` restarts without warnings |
| `restart.warning-interval-seconds` | `15` | Seconds between countdown warnings; values below 1 clamp to 1 |

### Log cleaner

| Key | Default | Description |
|---|---|---|
| `log-cleaner.enabled` | `true` | Enable log management |
| `log-cleaner.retention-days` | `7` | Delete log entries older than this |
| `log-cleaner.max-size-mb` | `500` | Maximum total `logs/` size; a negative value disables the size pass with a warning |
| `log-cleaner.keep-newest` | `5` | Newest files always kept during size cleanup |
| `log-cleaner.check-interval-minutes` | `30` | Cleanup check cadence in minutes; values below 1 clamp to 1 |

### messages.yml

All player- and console-facing text lives in `plugins/systemproxy/messages.yml` — filter block and kick messages, restart warnings, staff alerts, status/help/test output — with legacy `&` color codes and placeholders such as `{max}`, `{seconds}`, `{player}` and `{type}`. Every key has a hardcoded fallback, so a missing key never produces empty output. Both files reload atomically via `/systemproxy reload`; a failed or empty parse keeps the previous values.

## FAQ

**Does it connect to the System backend plugin?**
No. System Velocity is fully standalone by design: the plugin descriptor declares zero dependencies, and it uses no plugin-messaging channels, Redis, MongoDB or any other backend service. It is silent about whatever backend plugins are installed, and nothing in its configuration must match anything on your servers. The shared name is product-line branding only.

**What exactly happens at the daily restart?**
At `restart.time` in `restart.timezone` the proxy broadcasts countdown warnings (60 seconds by default, one every 15 seconds), sends a final "restarting now" message, and shuts the proxy down two seconds later — your host's process manager then brings it back up. An in-progress countdown is preserved across `/systemproxy reload`, and `restart.warning-seconds-before: 0` restarts with no countdown.

**Does the Unicode filter interfere with non-Latin alphabets?**
No. The filter targets genuinely malicious code points — zero-width characters, RTL/bidi overrides and isolates, control characters and invisible look-alikes. Normal Persian and Arabic script, standard punctuation and emoji pass through untouched; malicious code points are replaced with `?`. The restart timezone is configurable as well (default Asia/Tehran), which suits international and Iranian networks alike.

**Which platforms are supported?**
Velocity 3.3.0+ running Java 21 or newer. It is not compatible with Paper, Spigot or BungeeCord.

**Does it need a database or external services?**
No. There is no MongoDB, Redis, SQLite or HTTP dependency; the only shaded library (snakeyaml) is relocated inside the jar. The log cleaner manages only the proxy's own relative `logs/` directory — never backend server logs or arbitrary paths.

**Does it kick players for flooding or long messages?**
No. Kick-after-violations applies only to the Log4Shell filter (enabled by default, after 3 violations) and the Unicode filter (disabled by default, after 5). Flood, length and null-byte blocks deny the individual message only.

## Links

- Store: [store.hypeland.org/plugins/system-velocity](https://store.hypeland.org/plugins/system-velocity)
- Documentation: [store.hypeland.org/docs/reference/system-velocity](https://store.hypeland.org/docs/reference/system-velocity)
- Discord: [discord.gg/kEPvGp8Yz7](https://discord.gg/kEPvGp8Yz7)

---

<p align="center">© Nerotek Development. All rights reserved.</p>
