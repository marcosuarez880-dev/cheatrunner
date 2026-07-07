# CheatRunner — Changelog

## v0.15

- **Added a "Switch to Max Compatibility" prompt** when a cheat fails to enable due to an unverifiable address. Doesn't change any defaults — only offers the existing preset at the moment it would help.
- **Added trainer author credits** to the cheat menu ("Cheats By") for JSON, SHN, and MC4 files.
- **Fixed:** SHN/MC4 game names with an apostrophe (e.g. "Assassin's Creed") were truncated in the cheat menu.
- **Fixed:** cheats depending on a "Master Code" mod could resolve to the wrong address and fail silently, including a case where a dependent's own name confused master-code detection. Also fixes the dashboard's MISMATCH badge showing incorrectly for these cheats.
- **Fixed:** cheat/patch writes silently failed on firmwares 12.xx.
- **Fixed:** some valid cheats sharing a code-cave block with other mods were wrongly blocked as "wrong address."
- **Fixed:** cheat entries with a dropped leading hex zero (e.g. `"1"` instead of `"01"`) failed to parse and silently did nothing.
- **Added a network watchdog** that recovers faster after sleep/wake or Wi-Fi reconnects, and notifies you when the console's IP changes.
- **Added an OLED theme** — true black background and panels, and noticeably less glow/decorative lines than the default Dark theme.
- **Session logs are now saved to `/data/cheatrunner/logs.txt`**, not just kept in memory — makes it possible to check what happened after a crash or a restart wiped the in-app log view.
- **Fixed: "Shutdown Payload" could silently hang forever** instead of shutting down CheatRunner, if a cheat apply was stuck at the exact moment shutdown was requested. Shutdown now always completes within a couple seconds either way.
- **Enlarged the header logo** and made its glow match the active theme's color instead of always being red.
- **Fixed:** the Cheats/Repositories/Patches tabs in the trainer modal could render as oversized buttons on the PS5 browser.
- **Fixed:** a Master Code cheat could show a false "Conflicts with active mod" against its own dependent, blocking it from being enabled, when the dependent's own name also mentioned "Master Code."
- **Fixed:** cheats that require a Master Code showed a confusing "VERSION MISMATCH" (and couldn't be toggled) before the Master Code was enabled — now shows a clear "requires Master Code first" state instead.
- **Fixed:** a Master Code's own button showed a false "PARTIAL PATCH" once its dependent cheats were enabled and writing into its shared cave, even though everything was working correctly.
- **Fixed:** the Settings popup could fail to open/render on the PS5 browser the same way the trainer modal did.
- **Game icons are now cached by the browser** instead of being re-downloaded every time the dashboard loads — faster repeat loads, especially on the PS5 browser.
- **Added HTTP keep-alive**, so the dashboard's background polling reuses one connection instead of opening a new one for every single request.
- **Settings theme dropdown now shows proper names** ("Dark", "Crimson", "Midnight", "OLED") instead of raw internal values.

<details>
<summary><b>v0.14</b></summary>

- **CheatRunner's home-screen tile now installs into Media Players instead of the Games library**, plus a follow-up `param.json` audit (badge type, locale key, DRM type) found by comparing PS5's own app database against a known Media-tab app.
- **Fixed:** dashboard updates (new buttons, PS5 perf fixes) weren't showing up on PS5s that had already loaded the page — the AppCache manifest wasn't being bumped every release, and no JS handler existed to actually swap in an updated cache when one was detected. Both fixed.
- **PS5 browser performance pass.** Removed Google Fonts (blocking external font load), added HTTP caching for static assets, replaced `backdrop-filter` blur with solid backgrounds, hid decorative background layers, converted several `box-shadow`/`clip-path` animations to cheap `opacity`/`transform`-only ones, disabled tile-stagger and a couple of scan-line animations during cheat toggle/launch, and stopped tile-selection from rebuilding the entire game grid on every click.
- **Migrated the memory read/write engine from ptrace to mdbg.** ptrace required stopping the whole game process for every read/write, and two threads attaching at once could corrupt an in-progress patch — this was the root cause of games freezing mid-cheat-apply. Confirmed fixed on real hardware. Added a CR3-walk fallback for firmware 8.20+, where Sony's native mdbg write call silently stops working.
- **Went fully mdbg-only in the write path** — `kernel_mprotect` and the old ptrace-based code-cave remap fallback were removed entirely from writes.
- **Full security/reliability audit (~22k lines).** Removed dead HTTP auth code that was never wired up, closed a shutdown race that could interrupt an in-flight write, and fixed several smaller bugs: an undefined-behavior shift in patch hex parsing, a duplicate constant, a signed-overflow bug in address math, an unguarded config read, and cross-module false positives in conflict detection.
- **Fixed two real kernel-panic/console-freeze incidents**, both reproduced on hardware and traced to `kernel_mprotect` itself (not page-protection state) corrupting kernel memory on certain addresses. `kernel_mprotect` was removed from both the write path and the execute-only-memory read fallback; the patch mask-pattern scanner no longer scans past the first unreadable memory chunk.
- **Fixed a rapid-toggling freeze**, where a stalled PS5 notification call could lock up the entire cheat-apply subsystem because it ran while holding the apply lock. Also reduced the HTTP server's per-request memory allocation from a flat 4 MB to a small buffer that grows only when needed.
- Added Maffioh to the Support modal's "Special thanks" list.
- Bumped to **0.14**.

</details>

<details>
<summary><b>v0.13</b></summary>

- **Fixed:** cheats/patches targeting execute-only game code (`.text`) always showed as unreadable/mismatched, even when correctly applied. Reads now temporarily unlock the page, read, and restore its original protection. (This approach was later replaced in v0.14 for stability reasons.)
- **Added a crash log.** SIGSEGV/SIGBUS are now caught and appended to `/data/cheatrunner/crash.log` instead of the daemon dying with no trace.
- **Added a Thermals & Fan panel** — live CPU/SoC/NVMe temperatures, a manual fan-on threshold slider (persisted across launches/reboots), and a °C/°F toggle.
- **Added a User Profile panel** — rename the active user and change their profile picture directly from the dashboard.
- **CheatRunner now installs its own home-screen tile automatically** on first boot.
- **Fixed:** "Memory Patch — Not Ready" was shown even on a fully working jailbreak — a status field was missing from the API response the dashboard actually reads.
- **Fixed two off-by-one bugs** in process-list scanning that could make CheatRunner miss its own process or the running game's pid on certain kernel versions.
- **Fixed:** payload redeploys could fail to reclaim the HTTP port from a stuck previous instance; the port is now released immediately instead of waiting on the old process.
- Removed two long-dead, riskier write functions in favor of the one path everything already used; upgraded the address-guessing heuristic from checking 1 instruction byte to 2, cutting false positives significantly.
- Live download progress for remote cheat sources; a warning toast when results come from a stale cached index.
- Collapsible, reorderable sidebar panels; fonts (Space Grotesk/Inter/JetBrains Mono) that were configured but never actually loading now load correctly.
- The running game is now pinned to the top of the game grid; tiles show playtime or last-played date; profile picture upload gained a crop/pan/zoom step before saving.
- Supporter credits added to the Support modal.

</details>

<details>
<summary><b>v0.12</b></summary>

- **Fixed `bind() failed: 48` dashboard disconnects.** CheatRunner tried to kill a previous instance of itself *before* escalating its own privileges, so the kill was silently rejected and the old instance kept holding the port forever. The kill now happens after escalation, and failed binds retry with a clear, bounded message instead of spamming forever.
- **Ptrace stability audit** — fixed a credential-elevation race across three threads that could leave CheatRunner's process permanently over- or under-privileged, a competing attach between the game monitor and an in-progress cheat apply, and an address-cache buffer size bug that silently dropped the entire learned-address cache.
- **Fixed "CheatRunner is busy" / "not responding" under game load** (mainly PS4 BC titles) — dashboard polling could spawn enough hung ptrace-attach threads to exhaust the server's thread pool. Added liveness checks, skips polling while an apply is in progress, and shortened timeouts.
- **Fixed a crash from concurrent thumbnail requests** — SQLite is compiled non-thread-safe in this build, and loading many game icons at once could corrupt its internal state and crash the whole daemon. Now serialized behind a lock.
- **Fixed the HTTP server not reconnecting after PS5 sleep/wake** and a related crash-detection gap where a dead game process could still be reported as running.
- **MC4/SHN address-resolution and cave-writing bugs, several fixed the same week:** a poisoned address cache could brick every mod in a cheat file instead of self-healing; code-cave pages were stripped of write permission after a cheat was applied, freezing the game the next time it wrote to that page; multiple "Master Code" groups in one file could resolve against the wrong group; a scan fallback could compute an address missing its module base entirely, landing on arbitrary memory; and the `allow_legacy_*` "I accept the risk" settings didn't fully apply to every guard that should have respected them.
- **Added an x86-instruction heuristic** to auto-pick the correct address for SHN/MC4 cheats with no verified baseline — reads a few candidate bytes and picks whichever looks like real code, instead of guessing.
- **Added an address-learning cache** so resolved addresses don't need to be re-probed on every apply.
- **Enabled code-cave fallback and master-code auto-fixup by default**, and added cave-overwrite/JMP-redirect support so switching between mutually-exclusive cheats (e.g. Speed Normal/3x/5x) no longer requires a game restart.
- **Patch engine hardening:** the per-patch line cap was silently truncating large patches (some 60 FPS patches have 100+ lines) — raised from 64 to 640 lines. Fixed a kernel panic and a separate SIGILL crash in the PS4 BC patch-write and XML-parsing paths, added full backup/rollback (all-or-nothing) with hard failure on verify mismatch, and blocked applying patches with unsupported or zero usable lines instead of silently no-op'ing them.
- **Patch engine now supports multiple XML files per title** (including a new `elf-arsenal` search path), with stable IDs so applied-state survives rescans.
- **Dashboard overhaul:** animated tab switching, refreshed icons/buttons/focus states, loading skeletons, a Favorites/Recents system with first-run onboarding, and a full Settings redesign — one-click Safe/Max Compatibility/Debug presets, plain-language labels, risk warnings on dangerous toggles, search, and reset-to-defaults.
- Smaller fixes: shutdown now uses SIGKILL instead of a graceful exit that could hang; crash suspects and disabled-cheat state now persist across restarts; a concurrent-write race could corrupt cached files (icons, address cache) sharing the same temp-file name; game version detection now checks more paths and no longer probes a kernel path that could panic older firmware; PlayStation Store lookup as a last-resort title-name source.

</details>

<details>
<summary><b>v0.11</b></summary>

- **Reduced log noise significantly** — several very chatty per-write and per-poll log lines were downgraded to debug level or throttled to fire only on change, cutting routine log volume by roughly 5–10x.
- **Split the dashboard's HTML/CSS/JS into separate files** served at their own routes, shrinking the main HTML shell from ~3000 lines to under 200.
- **Fixed PS4 BC cheat writes failing silently** — added a fallback write path for pages where the standard permission query isn't supported.
- **Version detection improvements** — now checks more SFO/param paths (including update/patch and external-drive locations) and warns once per game instead of spamming when a version genuinely can't be found.
- **Added a launch-status generation counter** so a stale, slow-to-complete launch attempt can no longer overwrite the status of a newer one; added a watchdog that auto-recovers a launch stuck in "busy" past a timeout.
- **Centralized cheat-file version matching** into one shared comparison function, fixing several inconsistent-match edge cases (leading zeros, trailing `.0` segments, unparseable strings silently "matching").
- **Added manual cheat-file selection** — when auto-selection doesn't pick the right version, you can now force a specific candidate file, with a visible mismatch warning if you do.
- **Compatibility:** added `/data/etaHEN/cheats` and `/data/elf-arsenal/cheats` as cheat search paths.
- **Cheat engine stability fixes** — a fast double-attach race, a stuck "no cheat file" error after a game restart, and a false "bytes mismatch" that blocked disabling MC4/SHN cheats with no explicit expected-bytes baseline.
- **Added remote cheat source downloads** — cheats can now be fetched automatically from configured repositories on game launch.
- **Added a SQLite-backed game list**, title-name resolution, playtime/activity tracking, and a redesigned sliding ON/OFF toggle switch with proper mixed/error/applying states.

</details>

<details>
<summary><b>v0.1</b></summary>

- Initial release
- ptrace-based memory write engine (`pt_copyin` / `pt_copyout`)
- Cheat format support: JSON, SHN/XML, MC4 (AES-256-CBC encrypted XML)
- Basic cheat enable / disable via HTTP API
- Game process detection and monitoring

</details>
