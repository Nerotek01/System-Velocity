<p align="center">
  <img src="https://img.shields.io/badge/version-2.2.1-blue?style=for-the-badge" alt="Version">
  <img src="https://img.shields.io/badge/platform-velocity%203.3.0%2B-informational?style=for-the-badge" alt="Platform">
  <a href="https://discord.gg/YOUR_REAL_INVITE_CODE"><img src="https://img.shields.io/badge/support-24%2F7%20discord-5865F2?style=for-the-badge&logo=discord&logoColor=white" alt="Discord"></a>
  <a href="https://mc.hypeland.org"><img src="https://img.shields.io/badge/demo-mc.hypeland.org-orange?style=for-the-badge" alt="Demo Server"></a>
</p>

<h1 align="center">System-Velocity</h1>

<p align="center"><strong>Proxy-level security and maintenance for Velocity networks.</strong></p>

# System-Velocity

**Production-grade proxy protection for Velocity 3.3.0+ (Java 21).**
Engineered for networks that demand multi-layer exploit defense, automated maintenance, and zero-downtime log management.
A single JAR protects your entire proxy layer — filtering Log4Shell payloads, malicious Unicode injection, chat flooding, oversized payloads, and null-byte exploits — while scheduling automated daily restarts and keeping your log files under control. Every filter uses pre-compiled regex, lock-free data structures, and O(1) sliding-window algorithms to ensure zero impact on proxy throughput.

---

## Table of Contents

- [Test Server – Try Before You Buy](#test-server--try-before-you-buy)
- [Why System-Velocity?](#why-system-velocity)
- [Competitive Comparison](#competitive-comparison)
- [Filter Pipeline – How It Works](#filter-pipeline--how-it-works)
- [Core Features – Expanded](#core-features--expanded)
- [Commands & Permissions](#commands--permissions)
- [Full Configuration](#full-configuration)
- [Frequently Asked Questions](#frequently-asked-questions)
- [Support & Purchasing](#support--purchasing)

---

## Test Server – Try Before You Buy

A live, fully functional demo network is available so you can evaluate System-Velocity before making any commitment.

```
IP: mc.hypeland.org
```

The test server runs the latest stable build with every filter enabled. You can experience the anti-exploit protection, restart scheduler, and log cleaner exactly as they would run on your own network. **No registration, no whitelist — connect and play immediately.**

---

## Why System-Velocity?

System-Velocity is not a basic chat filter strapped to a proxy. It is a purpose-built security and maintenance layer that sits between your players and your backend servers, eliminating entire classes of attacks before they ever reach Spigot.

### The Only Proxy Plugin That Covers Everything
Every protection you need — Log4Shell filtering with multi-pass nested obfuscation detection, Unicode injection blocking with full code-point coverage, chat flood prevention with sliding-window algorithms, message length enforcement, and null-byte blocking — lives inside **one JAR**. You never install separate security plugins, you never configure separate filter chains, and you are never left debugging overlap conflicts between half a dozen proxy mods.

### Performance as the Highest Priority
All filter operations use pre-compiled regex patterns stored in `Pattern` constants — they are compiled once at startup and reused on every message, so there is zero regex compilation overhead at runtime. Flood tracking uses `ArrayDeque` with O(1) amortized insert and evict. Shared state uses `ConcurrentHashMap` for lock-free reads. Configuration maps are volatile for safe hot-reload. The restart scheduler is fully synchronized so reload, countdown, and shutdown operations can never interleave unsafely. The result is a plugin that adds **near-zero latency** to every chat message passing through the proxy.

### Battle-Tested Exploit Detection
The Log4Shell filter performs multi-pass analysis: direct JNDI lookup detection, curly-brace syntax flattening up to 4 levels deep to catch nested obfuscation like `${${lower:j}ndi:ldap://evil.com}`, dangerous lookup pattern detection, payload scheme detection (ldap, ldaps, rmi, dns, iiop, nds, nis, corba), and fail-safe handling of oversized payloads. Any message longer than 4096 characters containing lookup syntax is blocked outright, eliminating truncation-based bypasses.

### Automatic Maintenance That Respects Your Players
The daily restart scheduler uses configurable timezone-aware timing with countdown warnings at configurable lead times and intervals. An in-progress countdown is preserved when the configuration is reloaded, so a reload can never silently cancel an imminent restart. All countdown tasks are tracked and cancelled on shutdown to prevent task leaks. The log cleaner runs asynchronously on startup and on a configurable interval, deleting old logs by retention period and by total size, without ever touching symbolic links.

### Zero Recurring Costs, True Unlimited License
One payment grants you a permanent license that covers **all proxies you own**. There are no monthly fees, no per-proxy charges, and no hidden costs.

### Direct Access to the Developer
When your proxy has an issue at peak time, you do not file a ticket and wait. You speak directly to the person who wrote the code. Support is available **24/7** through Discord or Bale.

---

## Competitive Comparison

| Criteria | **System-Velocity** | VelocityScripts | TCPShield | Other "Premium" Proxy Plugins |
|----------|---------------------|-----------------|-----------|------------------------------|
| **Log4Shell filter** | Multi-pass (4-level nesting, 9+ schemes, 4096-char fail-safe) | Basic or none | Differently scoped (network-level) | Rare |
| **Unicode injection filter** | Full code-point coverage (zero-width, RTL, control, surrogate pairs) | Not available | Not available | Rare |
| **Chat flood filter** | Sliding-window O(1) with configurable window and cooldown | Basic rate limit | Not available | Varies |
| **Null-byte filter** | Dedicated first-in-pipeline check | Not available | Not available | Rare |
| **Violation tracker** | Per-player, per-type, periodic sweep, configurable kick | Basic | Not available | Varies |
| **Broadcast alerts** | Staff alerts with safe placeholder interpolation | Not available | Not available | Rare |
| **Daily restart** | Timezone-aware with preserved countdown on reload | Not available | Not available | Rare |
| **Log cleaner** | Async, size + age based, symlink-safe, NIO.2 | Not available | Not available | Rare |
| **Self-test system** | Built-in `/systemproxy test` for filter verification | Not available | Not available | Not available |
| **Configuration** | YAML with malformed-file hardening and safe defaults | Varies | Web dashboard | Varies |
| **License model** | Permanent, all proxies | Free / recurring | Free (with limits) | Often per-proxy or recurring |

**Key takeaway:** System-Velocity is the only plugin that combines multi-layer exploit filtering, automated maintenance, and log management in a single JAR for your Velocity proxy — with one purchase that covers every proxy you run.

---

## Filter Pipeline – How It Works

When a player sends a chat message, the `ChatListener` processes it in the following order (first match blocks the message):

1. **Null-byte check** — Blocks any message containing `\0`, which can truncate strings in backend C-based parsers.
2. **Flood check** — Sliding-window algorithm tracks message timestamps per player in an `ArrayDeque` with O(1) amortized insert and evict.
3. **Length check** — Enforces a maximum message length (default 256 characters) to prevent oversized payload attacks.
4. **Log4Shell exploit check** — Multi-pass analysis: direct JNDI detection, 4-level curly-brace flattening, dangerous pattern detection, 9+ scheme validation, and 4096-char fail-safe.
5. **Unicode injection check** — Blocks zero-width characters, RTL/bidi formatting, control characters, suspicious invisible characters, with proper surrogate-pair handling.

Each check can independently block the message, notify the player, and log the event. The Log4Shell and Unicode checks also feed into the **ViolationTracker**, which can trigger a kick after repeated violations. All shared state uses `ConcurrentHashMap` for thread-safe access without global locks.

---

## Core Features – Expanded

### Anti-Log4Shell Filter

Detects and blocks Log4Shell (CVE-2021-44228) exploit attempts in chat messages. The filter performs multi-pass analysis across the entire message:

| Capability | Detail |
|------------|--------|
| **Direct JNDI detection** | Case-insensitive across the entire message |
| **Nested obfuscation** | Curly-brace flattening up to 4 levels deep (e.g. `${${lower:j}ndi:ldap://...}`) |
| **Dangerous patterns** | `lower`, `upper`, `env`, `sys`, nested braces, double-colon bypasses |
| **Scheme detection** | ldap, ldaps, rmi, dns, iiop, nds, nis, corba |
| **Oversized payload fail-safe** | Messages >4096 chars with lookup syntax are blocked outright |
| **Log sanitization** | Neutralizes `${` sequences before blocked content reaches the console |

### Anti-Unicode Injection Filter

Blocks malicious Unicode characters used for obfuscation, spoofing, or log injection:

| Category | Examples |
|----------|---------|
| **Zero-width characters** | U+200B, U+200C, U+200D, U+FEFF, U+2060, U+180E |
| **RTL/bidi formatting** | U+202A–U+202E, U+200E, U+200F, U+2066–U+2069 |
| **Control characters** | All except tab, newline, carriage return |
| **Invisible characters** | Soft hyphen, combining grapheme joiner, Arabic letter mark, Hangul fillers, variation selectors, zero-width joiners |
| **Surrogate pairs** | Full Unicode code point coverage for modern clients |

Normal Persian, Arabic script, standard punctuation, and emoji pass through untouched.

### Anti-Flood Filter

Prevents chat flooding using a sliding-window algorithm with O(1) amortized complexity:

- Configurable window size (default 3 seconds) and maximum messages per second (default 5).
- Automatic cooldown period when threshold is exceeded (default 10 seconds).
- Thread-safe per-player locking using synchronized blocks on the player's own deque.
- Overflow-safe limit computation prevents wrap-around exploits.

### Length & Null-Byte Filters

- **Length filter**: Enforces maximum message length (default 256) after flood check, before exploit checks.
- **Null-byte filter**: Blocks any message containing `\0`, running first in the pipeline.

### Violation Tracker

Tracks security violations per player and per type with memory-safe design:

- `AtomicInteger` for thread-safe violation increments.
- Automatic clearing on player disconnect (configurable).
- Periodic 5-minute retention sweep bounds memory to the online player set.
- Manual clearing via `clearAll()` on proxy shutdown.

### Broadcast Alerts

Optional staff alerts with safe placeholder interpolation that prevents color-code injection through crafted usernames on offline-mode proxies.

### Daily Restart Scheduler

- Configurable timezone (default Asia/Tehran) and time in HH:mm format (default 04:00).
- Countdown warnings with configurable lead time and interval.
- In-progress countdown preserved across configuration reloads.
- All tasks tracked and cancelled on shutdown to prevent task leaks.

### Log Cleaner

- Runs asynchronously on startup and on a configurable interval (default 30 minutes).
- Deletes logs older than retention period (default 7 days).
- Size-based cleanup with configurable maximum (default 500MB) and keep-newest count (default 5).
- Symbolic links are never created, followed, or deleted.
- Uses NIO.2 file tree API with try-with-resources.

---

## Commands & Permissions

| Command | Description | Permission |
|---------|-------------|------------|
| `/systemproxy reload` | Reload configuration, reschedule restart, refresh log cleaner | `systemproxy.admin` |
| `/systemproxy status` | Show current status of all filters | `systemproxy.admin` |
| `/systemproxy test` | Run built-in filter self-tests (Log4Shell + Unicode) | `systemproxy.admin` |
| `/systemproxy help` | Show help message | `systemproxy.admin` |

Aliases: `/sp`, `/sysproxy` — Full tab-completion for subcommands.

| Permission | Description | Default |
|------------|-------------|--------|
| `systemproxy.admin` | Access to all `/systemproxy` subcommands | OP only |
| `systemproxy.alerts` | Receive broadcast alerts for violations | OP only |

---

## Full Configuration

All configuration is stored in `plugins/systemproxy/config.yml`. All player-facing messages are customizable in `plugins/systemproxy/messages.yml` with legacy ampersand color codes.

### Log4Shell Filter

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `log4shell.enabled` | boolean | `true` | Enable the Log4Shell filter |
| `log4shell.notify-player` | boolean | `true` | Notify the player when blocked |
| `log4shell.log-blocked` | boolean | `true` | Log blocked attempts to console |
| `log4shell.kick-enabled` | boolean | `true` | Kick after exceeding max violations |
| `log4shell.max-violations` | int | `3` | Violations before kick |

### Unicode Filter

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `unicode-filter.enabled` | boolean | `true` | Enable the Unicode filter |
| `unicode-filter.notify-player` | boolean | `true` | Notify the player when blocked |
| `unicode-filter.log-blocked` | boolean | `true` | Log blocked attempts |
| `unicode-filter.kick-enabled` | boolean | `false` | Kick after exceeding max violations |
| `unicode-filter.max-violations` | int | `5` | Violations before kick |

### Flood Filter

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `flood-filter.enabled` | boolean | `true` | Enable the flood filter |
| `flood-filter.max-messages-per-second` | int | `5` | Maximum messages per second |
| `flood-filter.window-seconds` | int | `3` | Sliding window size in seconds |
| `flood-filter.cooldown-seconds` | int | `10` | Cooldown duration after flooding |
| `flood-filter.notify-player` | boolean | `true` | Notify the player when blocked |
| `flood-filter.log-blocked` | boolean | `true` | Log blocked attempts |

### Length & Null-Byte Filters

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `length-filter.enabled` | boolean | `true` | Enable the length filter |
| `length-filter.max-length` | int | `256` | Maximum allowed message length |
| `null-byte-filter.enabled` | boolean | `true` | Block messages containing null bytes |

### Alerts

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `alerts.broadcast` | boolean | `false` | Broadcast violations to staff |
| `alerts.auto-clear-violations-on-quit` | boolean | `true` | Clear violation data on disconnect |

### Restart Scheduler

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `restart.enabled` | boolean | `true` | Enable daily restart |
| `restart.time` | string | `"04:00"` | Restart time (HH:mm) |
| `restart.timezone` | string | `"Asia/Tehran"` | Timezone for restart time |
| `restart.warning-seconds-before` | int | `60` | Seconds of warnings before restart |
| `restart.warning-interval-seconds` | int | `15` | Interval between warnings |

### Log Cleaner

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `log-cleaner.enabled` | boolean | `true` | Enable the log cleaner |
| `log-cleaner.max-size-mb` | long | `500` | Maximum total log size (MB) |
| `log-cleaner.keep-newest` | int | `5` | Newest log files to always keep |
| `log-cleaner.retention-days` | int | `7` | Delete logs older than this |
| `log-cleaner.check-interval-minutes` | long | `30` | Check interval in minutes |

---

## Frequently Asked Questions

### General Questions

**Q: Which proxy platforms are supported?**
A: System-Velocity is built exclusively for Velocity 3.3.0+ (Java 21). It is not compatible with BungeeCord.

**Q: Does it work alongside backend anticheat plugins?**
A: Yes. System-Velocity operates at the proxy layer, filtering chat before it reaches backend servers. It complements any backend Spigot anticheat.

**Q: How does the license work?**
A: One payment of **€5.00** grants you a permanent license that covers every proxy you own. There are no recurring fees, no per-proxy charges, and no hidden costs.

**Q: Can I test the plugin before buying?**
A: Yes. Connect to `mc.hypeland.org` to experience the full plugin on a live network.

### Technical Questions

**Q: What Java version does my proxy need?**
A: Java 21 or newer.

**Q: What happens if I reload the configuration during a restart countdown?**
A: The in-progress countdown is preserved. A reload can never silently cancel an imminent restart.

**Q: Does the log cleaner follow symbolic links?**
A: No. Symbolic links are never created, followed, or deleted, so linked files outside the logs directory are always safe.

**Q: How does the self-test system work?**
A: The `/systemproxy test` command runs a suite of test cases against the Log4Shell and Unicode filters, reporting PASS/FAIL for each case and a summary at the end.

### Support

**Q: How do I get help if something breaks?**
A: You have 24/7 direct access to the developer via Discord (`Nerotek01`) or Bale (`Nerotek`). There are no tickets, no forums, and no canned replies.

**Q: Are updates free?**
A: All updates for the current major version are included with your permanent license.

---

## Support & Purchasing

**System-Velocity** is a premium plugin sold exclusively by the developer.

### How to Purchase
- **Discord:** `Nerotek01`
- **Bale (Iranian users):** `Nerotek`
- **Price:** **€5.00** — one-time payment, permanent license.

### License
**Permanent, all-proxies license.** Your purchase covers every proxy you own. There are no recurring fees, no per-proxy charges, and no hidden costs.

### What You Receive
- The complete System-Velocity plugin JAR.
- All security filters, fully integrated and ready to use.
- Daily restart scheduler with timezone-aware countdown.
- Log cleaner with async size and age management.
- Built-in self-test system for filter verification.
- Free updates for the current major version.
- **24/7 priority support** via Discord or Bale.

### Support Promise
When an issue arises on your live network, you do not file tickets and hope for a reply. You speak directly with the developer — the person who wrote every line of code. Your uptime is our reputation.

---

<p align="center">
  <a href="https://mc.hypeland.org"><strong>Connect to the demo: mc.hypeland.org</strong></a>
</p>
