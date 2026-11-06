# RCON Tool Companion

**Version:** 1.0.3 | **Author:** ShAgGy

RCON Tool Companion (RCT) is a standalone CounterStrikeSharp plugin for CS2 Fun Stuff modes and player-level commands. It includes its own in-game administration menu and does not depend on SimpleAdmin or a separate menu plugin.

The [Ultimate CS2 RCON/Server Tool](https://github.com/ShAgGy2035/Ultimate-RCON-Server-Tool) is optional. Use RCT alone with `!fun`, or use the desktop tool for mode settings, scheduled rotations, RCON commands, and server management.

See the [1.0.3 release notes and downloads](https://github.com/ShAgGy2035/RconCompanionTool/releases/tag/v1.0.3) for the current release.

## Contents

- [Requirements](#requirements)
- [Install and verify](#install-and-verify)
- [Start a mode](#start-a-mode)
- [Available modes](#available-modes)
  - [Workshop maps and Surf map voting](#workshop-maps-and-surf-map-voting)
  - [Bot balancing in round-based team play](#bot-balancing-in-round-based-team-play)
  - [Bot Control over an active mode](#bot-control-over-an-active-mode)
- [Weapon Rotation](#weapon-rotation)
- [Super Hero: all fourteen powers](#super-hero-all-fourteen-powers)
- [One in the Chamber](#one-in-the-chamber)
- [Bot settings](#bot-settings)
- [Rollback and map changes](#rollback-and-map-changes)
- [Configuration and upgrades](#configuration-and-upgrades)
  - [Fixed-value JSON reference](#settings-with-fixed-values-hand-editing-rctjson)
- [Command reference](#command-reference)
- [Player targets and examples](#player-targets-and-examples)
- [Managed player state](#managed-player-state)
- [Troubleshooting and validation](#troubleshooting-and-validation)

## Requirements

- A Counter-Strike 2 dedicated server.
- Metamod:Source.
- The **full CounterStrikeSharp 1.0.376 runtime**, or a compatible newer runtime. Expanded Super Hero powers use native pre-damage listeners and hull traces; replacing an older runtime's API DLL alone is not sufficient.

RCT targets .NET 10 and CounterStrikeSharp API 1.0.376. MenuManagerAPI, MenuManager, and PlayerSettingsCS2 are not required; they may remain installed for other plugins.

## Install and verify

1. Back up the existing plugin and `configs/plugins/RCT` directory before upgrading. Download and extract `RconToolCompanion-v1.0.3.zip` into the server's `game/csgo/` directory. The archive starts at `addons/`.
2. Confirm the deployed plugin is at:

   ```text
   game/csgo/addons/counterstrikesharp/plugins/RCT/RCT.dll
   ```

3. Restart the server or load RCT through CounterStrikeSharp.
4. Run `css_rct_status` from the server console or RCON. A successful response starts with `[RconToolCompanion] Ready.` and reports the version and capabilities.
5. Grant trusted in-game administrators the CounterStrikeSharp permission `@css/fun`.

If PlayerPunishments is installed, upgrade it to 1.0.1 alongside Companion 1.0.3. Companion now provides `css_alive`; an older PlayerPunishments copy would register the same command.

The short `RCT` directory is intentional: affected Linux container filesystems can fail to canonicalize a longer plugin path. Keep the deployed directory and DLL named `RCT` and `RCT.dll`.

## Start a mode

### In game

On a test server, type `!fun` or `/fun` in chat with the `@css/fun` permission. Start with one mode and verify rollback before using it in a public game.

| Key | Action |
| --- | --- |
| W / S | Move through options |
| E | Select |
| A | Return to the previous menu |
| R | Exit |

Administrative menus hold the player in place while open; public category-vote popups allow movement.

1. Select a gameplay mode, then optionally customize gameplay or bots.
2. Return with **Done** and select **START MODE**.
3. Reopen `!fun` and choose **Rollback** when finished to restore the captured server settings.

For example, open **Bhop > Customize gameplay**, change an option, return with **Done**, then start. Cancelling leaves the running mode unchanged. Reopening a wizard starts from configured defaults, not your previous temporary choices.

**Mode customization is temporary:** it lasts for the current run, including map changes, but never saves to `RCT.json`. Reload/restart recovery uses configured defaults instead. To change permanent defaults, follow [Configuration and upgrades](#configuration-and-upgrades). The old editable menu and **Save settings only** are removed; `UseFunSetupWizard` is ignored, even when `false`.

**No Bots** starts directly. The separate **Bot control** panel uses **APPLY BOT SETTINGS** and still saves its own preferences.

Administrative menus close after ten minutes without navigation and restore movement. If permission is revoked, the menu closes and unfinished edits are discarded. Reopen `!fun` after access is restored. Public category voting does not require administrator permission.

In-game administrators do not need an RCON password or the desktop app.

### Server console or RCON

Use `css_rct_fun_apply <mode>` to start a complete mode using its `RCT.json` defaults:

```text
css_rct_fun_apply superhero
css_rct_status
css_rct_mode_rollback
```

Administrative console commands are server/RCON-only. The lower-level `css_rct_mode_enable` selects a profile; it is not a substitute for the complete configuration sequence. For ordinary starts and scheduled rotations, use `css_rct_fun_apply`.

### Optional desktop integration

Universal exposes the Fun Stuff settings, including all fourteen Super Hero toggles and strengths. Universal and `!fun` can replace or roll back modes started by either interface. RCT owns their shared rollback snapshot; each gameplay mode retains independent rules and handlers.

Choose the mode and its settings in Universal's Fun Stuff area, or create a scheduled rotation there. Map carryover depends on which interface owns the mode; see [Rollback and map changes](#rollback-and-map-changes) before leaving the desktop app closed.

## Available modes

Use the token below with `css_rct_fun_apply`. In `!fun`, No Bots and Bot control appear first, gameplay modes follow alphabetically, and Rollback is last.

| Mode | Token | Overview |
| --- | --- | --- |
| No Bots | `nobots` | Sets bot quota to zero and removes bots without changing weapons, buying, or ground cleanup. |
| AWP / Sniper Wars | `awp` | Managed AWP-only loadouts; configurable forced loadout and infinite ammo. |
| Bhop | `bhop` | Bunnyhopping and optional stamina-limit removal, with unrestricted weapons. |
| Deathmatch | `deathmatch` | Configurable respawns, random spawns, duration, ammo, bot AI, and kill-streak medikit rewards. |
| Flying Dutchman | `flying-dutchman` | Configurable movement, bunnyhopping, weapon accuracy, ragdoll gravity, and Workshop maps. |
| Grenade Wars | `grenades` | Managed HE grenades with replenishment after throws. |
| Headshot Only | `headshot` | Headshot rules with unrestricted weapons. |
| Knife Arena | `knife` | Managed team-knife loadouts. |
| Molotov / Incendiary Wars | `fire-grenades` | Managed team-appropriate fire grenades with replenishment. |
| One in the Chamber | `oneinthechamber` | Native free-for-all with a Deagle, knife, and one starting bullet. |
| Pistols Only | `pistols` | Managed team-default pistol loadouts, with optional headshots-only rules. |
| ScoutzKnives | `scoutzknives` | SSG 08 and team knife, with configurable movement and physics. |
| Super Hero | `superhero` | One of fourteen enabled powers per life; normal weapons and buying. |
| Surf | `surf` | Configurable surf physics, stamina, round time, and freeze time. |
| Weapon Rotation | `weapon-rotation` | Rotating firearms with category voting, announcements, and independent gameplay settings. |
| Zeus Wars | `zeus` | Managed Zeus-only loadouts. |

**Bot Control** is a separate panel, not a replacement gameplay mode. It can adjust bots while another mode remains active.

ScoutzKnives exposes movement, gravity, bunnyhopping, fall-damage scale, and friction. Surf also exposes acceleration, jump/landing stamina costs, round time, and freeze time without changing `sv_cheats`. Flying Dutchman's optional ragdoll gravity changes only the relevant cvar's cheat flag, not global `sv_cheats`. Grenade Wars, fire-grenade modes, and Zeus Wars always enforce their required loadouts; they have no optional `ManagedLoadout` switch.

### Workshop maps and Surf map voting

Supported modes have their own start map, Workshop map ID or collection ID, and optional GG1MapChooser pool settings. Configure these in Universal or the individual mode's `RCT.json` section; map editing is not part of the in-game wizard.

Surf's separate `MapTimeLimitMinutes` defaults to 30. When the limit is reached, RCT asks GG1MapChooser to vote using the Surf pool and change maps; `0` disables this timer. It restarts on each map and stops on rollback, without changing `mp_timelimit` or `mp_maxrounds`. Install a compatible GG1MapChooser to use that voting workflow.

When pool swapping is enabled, RCT backs up the chooser's map list and restores it when the mode ends. Surf rollback can return to the pre-mode map; switching directly to another mode keeps the current map. Test Workshop downloads, voting, and map return with your installed plugins before using them publicly.

### Bot balancing in round-based team play

ScoutzKnives, Pistols Only, Knife Arena, Headshot Only, AWP / Sniper Wars, Bhop, Grenade Wars, Molotov / Incendiary Wars, Zeus Wars, Surf, Flying Dutchman, and Super Hero each have their own independent bot-balancing handler.

- Set `mp_autoteambalance 1` in the server configuration to enable native balancing and these bot-only checks; `0` disables the checks. No extra per-mode checkbox is required.
- At round starts, eligible bots move from the larger team until the difference is at most one. This also covers the only human spectating a bot game. The fallback never moves humans or bots under human control; if no eligible bot exists, the imbalance is left unchanged.
- Checks skip warmup, halftime/team switching, match-end screens, fixed-team Bot Control, FFA, and gameplay with respawns or ignored round-win conditions. Continuous-respawn modes remain unchanged; Deathmatch, One in the Chamber, and No Bots have no new balancing handler.
- Startup, map reapplication, and rollback retain their existing settings. The checks read the current server state rather than changing cvars or keeping extra runtime/persisted settings. These apply equally to starts from `!fun`, Universal, or scheduled tasks.
- Optional RCT debug logging records per-mode team counts, confirmed bot moves, and any remaining imbalance without player identities. Missing game state or an unconfirmed move is reported in the server log.

Weapon Rotation keeps its separate **Auto-balance teams** option and independent implementation described below.

### Bot Control over an active mode

Use the in-game Bot control panel or Universal's Bot Control tab to adjust compatible modes without replacing them.

- In `!fun`, open Bot control, edit Bot count / Teams and halftime / Difficulty, then Apply. A returns to the previous menu. Mode-submenu bot settings remain independent startup defaults; **Adjust ACTIVE bots** opens the live layer without starting the selected mode.
- **Restore bots ONLY (keep mode)** returns to the bot setup captured before the first edit. **Rollback ALL (mode + bots)** restores the pre-mode setup. Human team restrictions are restored, but rollback does not move humans back to their former teams.
- Changing modes clears overrides. Map reapply preserves them while active; empty-server rollback restores both layers after the configured grace period.
- Halftime offers Keep current, Disable, and Enable. Keep current sends no halftime change. Fixed assignments remain tied to CT/T and are reconciled after automatic/manual side swaps; they do not follow the original group to the other side. Disable halftime to avoid automatic swaps.
- No Bots must be rolled back before using Bot Control. Deathmatch and One in the Chamber allow bot count/difficulty edits but block team restrictions and enabling halftime.
- Applying/restoring bots is blocked during another mode operation or map restoration. Failures are reported without claiming success.

Server/RCON command:

```text
css_rct_bot_control_apply <quota> <counting> <bot-team> <human-team> <one-team> <difficulty> <halftime>
```

Quota: `-1` keep, `0` off, or `1`-`64`. Counting: `keep`, `normal`, `fill`, `match`. Teams: `keep`, `any`, `ct`, `t`. One-team: `-1` keep, `0` off, `1` on. Difficulty: `-1` keep or `0`-`5` (both difficulty cvars are set). Halftime: `-1` keep, `0` disable, `1` enable. One-team requires enabled bots on CT or T and always uses fixed normal counting.

Use `css_rct_bot_control_rollback` for bots only; `css_rct_mode_rollback` restores everything, even when only Bot Control is active. The legacy `css_rct_bot_team` command now uses the same snapshot and compatibility checks.

Try team and halftime changes on a test server before applying them to a public game.

## Weapon Rotation

Start through `!fun > Weapon Rotation`, Universal, or a scheduled task. Choose from 34 firearms across six categories, with shared or individual assignments and team rounds, team respawn, or free-for-all respawn.

- **Weapons:** enable categories and select individual weapons. Both filters apply.
- **Interval:** 5-3600 seconds, default 300 (five minutes).
- **Change weapons:** choose Timed interval or Every round in Team rounds. Every round changes the assignment at the next round start after a completed round; respawns retain that round's assignment. It follows the existing server round length (including 1:55 rounds) without changing round-time cvars. Respawn gameplay supports timed rotation only. The saved seconds interval is ignored, not discarded, while rotating by round.
- **Duration:** `0` is unlimited; 1-1440 minutes limits the whole mode, then restores the mode and Bot Control settings. Universal/scheduled starts retain their original deadline across plugin reloads. A temporary `!fun` run instead discards its choices on reload and recovers using configured defaults, including the configured duration.
- **Equipment:** choose knives, grenades, armor, buying, pickups, and ammunition. Infinite ammunition defaults on without enabling `sv_cheats`.
- **Auto-balance teams:** on by default, configurable in Gameplay or `AutoBalanceTeams`. Enables native CS2 balancing and a one-player team-size limit. At round starts, a bot-only fallback moves eligible bots from the larger team until the difference is at most one, including when the only human is spectating. It never moves humans or bots under human control, and skips warmup, halftime/team switching, and match-end screens. It does not act immediately on spectator changes. Team respawn has no regular round boundaries, so this does not guarantee ongoing rebalancing there. FFA and fixed-team Bot Control setups remain unrestricted; bot-count/difficulty-only changes preserve your balance choice. Previous server settings are restored on mode rollback.
- **Announcements:** choose Popup, Chat, or both. Chat reminders default to 30 seconds; `0` disables periodic reminders. Menus and map voting take priority over popups. Weapon popups and automatic category menus stay hidden through halftime/team switching and the first second-half freeze period.
- **Category voting:** enabled by default for category rotation. A popup shows all six eligible categories without scrolling; `!wrvote` reopens it. Votes are selected in the popup, not typed in chat. Invitations and results still appear in chat.
- **Maps:** configure a start map or Workshop map/collection in Universal or this mode's `RCT.json` section. Map editing is not included in `!fun`.

With timed rotation, voting normally opens 30 seconds before a change and closes five seconds before it. Short intervals reduce or skip voting. With Every round, voting runs during the round and closes at round end; the winning category applies at the next round start. No votes follows normal rotation, and ties are chosen randomly. Weapons within the winning category remain automatic. Round-based notices say "Changes next round" rather than estimating a countdown; halftime/menu-priority suppression still applies.

Living players receive replacements when the interval advances; respawning keeps the current assignment. Team rounds retain bomb objectives. The mode continues after Universal closes.

For exact in-game values, first open Weapon Rotation in `!fun`, then use `!wrset interval 300`, `!wrset duration 30`, `!wrset reminder 30`, `!wrset respawn 0.5`, or `!wrset quota 10`. These edit only your temporary wizard draft and require `@css/fun`; select **START MODE** to apply them without saving. Other modes support `!funset <mode> <setting> <value>` on their open wizard. Use the [fixed-value reference](#settings-with-fixed-values-hand-editing-rctjson) for permanent JSON edits. An active run keeps its starting settings; roll back and restart to apply a new draft.

## Super Hero: all fourteen powers

Start **Super Hero** in `!fun`, through Universal, or with `css_rct_fun_apply superhero`. Normal weapons and buying remain available.

Each spawn receives **one enabled power**, not all fourteen at once. Powers are allocated from the least-used choices among living teammates, independently for CT and T. A five-player team gets five different powers when at least five are enabled; larger teams receive balanced repeats. Respawns prefer a different power when an equally suitable choice exists. Humans and bots have individual assignments.

All fourteen toggles default to `true`. Configure them inside the `SuperHero` section of `RCT.json`; defaults below apply to new or missing fields, not values already saved in your file.

| Power | JSON toggle | What it does / player control | Strength settings and defaults |
| --- | --- | --- | --- |
| Speedster | `Speedster` | Faster movement. | `SpeedMultiplier`: `1.6` |
| Moon Jumper | `LowGravity` | Hold Jump during takeoff for a high jump and slower descent. Fall damage is cancelled for this power. | `MoonJumperJumpVelocity`: `1200` units/s; `LowGravityScale`: `0.275` |
| Tank | `Tank` | Increased health and full armor. | `TankHealth`: `250` |
| Ghost | `Ghost` | Nearly invisible appearance. | `GhostVisibilityPercent`: `15` |
| Vampire | `Vampire` | Heals a percentage of damage dealt to other players. | `VampirePercent`: `50` |
| Regenerator | `Regenerator` | Heals after each enemy kill. | `KillHealAmount`: `50` |
| Juggernaut | `Juggernaut` | Reduced incoming damage at the cost of movement speed. | `JuggernautDamageReductionPercent`: `35`; `JuggernautSpeedMultiplier`: `0.75` |
| Second Wind | `SecondWind` | Survives one lethal incoming hit per life, restoring survival health. | `SecondWindHealth`: `35` |
| Ammo Recycler | `AmmoRecycler` | Enemy kills refill the active magazine. | No numeric setting. |
| Berserker | `Berserker` | Increased enemy damage and movement speed while below the health threshold. | `BerserkerHealthThreshold`: `35`; `BerserkerDamageMultiplier`: `1.5`; `BerserkerSpeedMultiplier`: `1.3` |
| Medic | `Medic` | Periodically heals nearby living teammates with line of sight, up to 100 health. | `MedicHealAmount`: `5`; `MedicRadius`: `300` units; `MedicIntervalSeconds`: `2` |
| Blink | `Blink` | Press Use (normally E) to attempt a forward teleport. Blocked paths, occupied destinations, and unsupported drops are rejected. | `BlinkDistance`: `250` units; `BlinkCooldownSeconds`: `8` |
| Double Jump | `DoubleJump` | Release Jump, then press it again while airborne for one extra jump. Landing resets it. | `DoubleJumpVelocity`: `250` units/s |
| Gunslinger | `Gunslinger` | Completes reloads sooner, transferring existing reserve ammo rather than creating ammo. | `GunslingerReloadMultiplier`: `0.5` of normal remaining reload time |

### Shared settings and input details

- `MaxHealth` defaults to `200` and controls Vampire/Regenerator healing caps; the cap is at least the pawn's maximum health.
- Moon Jumper's launch velocity accepts `300`-`1500` units/s. Gravity accepts `0.1`-`1`, where `1` is normal. The `0.275` default approximates ScoutzKnives' gravity of 220 on a standard 800-gravity server.
- To test a particular power, enable only that power; otherwise assignment selects among the enabled choices.
- Bots trigger Blink when an enemy is ahead and use Double Jump automatically while descending.
- The Fun Stuff menu suppresses power-input handling, so selecting with E does not activate Blink.
- Pending Gunslinger reload completion is cancelled when the weapon, life, power, or map changes.
- Rollback removes active powers. Map recovery resets power/life/cooldown state before assigning powers again.

### Power notices

`PowerNoticeSeconds` defaults to `5`, accepts `0`-`30`, and disables center-screen popups at `0`. Universal labels this **Power popup (seconds)**.

The notice shows the assigned power and the same description as chat, including activation instructions. Spectators see the living player or bot they are watching, with the subject's name and a bot label. A target, power, or life change starts a fresh notice.

Notices do not freeze movement or accept menu input. `!fun` takes priority and pauses an existing notice's clock; expiration, mode exit, and map reset clear it. Chat remains a fallback. RCT refreshes cached notice HTML each server tick to maintain its display; exact live duration and residual HUD dimming remain validation items.

Use `!power` or `css_power` to recall your current power and re-show its notice. While spectating, it shows the living player or bot you are watching; it also follows bot takeover. `SuperHero.PowerReminderSeconds` controls periodic chat reminders: default `30`, range `0`-`300`, with `0` disabling reminders. Set a session value with `css_rct_superhero_reminder <seconds>`.

## One in the Chamber

Start with `css_rct_fun_apply oneinthechamber`, `!fun`, or Universal.

- Each spawn grants the stock Desert Eagle, team knife, one bullet, and no reserve ammo.
- Each non-self player kill earns one bullet, including knife kills and kills on the same nominal team. Earned bullets can accumulate up to 64.
- Missing the shot leaves the knife as the way to earn another bullet.
- Buying, other weapons, map-placed weapons, and C4 objectives are disabled. A per-life ammo ledger prevents reloads, pickups, or bot takeovers from creating extra bullets.

| `OneInTheChamber` setting | Default | Meaning |
| --- | --- | --- |
| `HeadshotsOnly` | `false` | By default, Deagle and knife hits are lethal. When enabled, Deagle body hits do no damage; knife hits remain lethal. |
| `RespawnDelaySeconds` | `1` | Companion-controlled respawn delay, from `0.25` to `10` seconds. |

Standard bot settings also apply. Chamber owns its native Deathmatch FFA rules independently of the Deathmatch mode: `game_type 1`, `game_mode 2`, and `mp_dm_teammode 0`, with individual scoring and random spawns. Internal CT/T assignments do not enable team scoring.

Chamber clears `bot_stop`, uses the stock bot behavior tree (`mp_bot_ai_bt ""`), and allows humans and bots on either team. Entry and rollback reload the current map. Its guarded five-second map reapply adds no round restart; rollback restores captured cvars after map configs load before starting a pending replacement mode.

Roll back an active older Chamber session before upgrading. This mode requires `one-in-the-chamber` and `chamber-native-ffa` capabilities and the full compatible CounterStrikeSharp runtime.

## Bot settings

Every configurable gameplay mode has these independent bot options:

All Fun Stuff modes ship with bots disabled, including Weapon Rotation. This default applies to new settings or omitted `BotsEnabled` fields; explicitly saved values and active runs are preserved. Standalone Bot Control is unchanged.

| Setting | Default | Behavior |
| --- | --- | --- |
| `BotsEnabled` | `false` | When disabled, sets quota to zero and kicks current bots; quota and quota mode are ignored. |
| `BotQuota` | `null` | Preserves the current server quota. Set `1`-`64` to enforce a count. |
| `BotQuotaMode` | `fill` | Accepts `normal`, `fill`, or `match`. `fill` replaces bots as humans join to maintain the configured total quota. |

All quota modes, including standalone Bot Control, default to `fill`. Existing explicit `normal` or `match` selections are preserved. In a mode's `!fun` submenu, administrators can select the current quota or presets from 1 to 20.

### Standalone Bot Control

Changes in **Bot control** are staged until **APPLY BOT SETTINGS**. Apply saves preferences to `RCT.json` before changing the live server; a save failure leaves the server unchanged. Enabling bots with a zero current quota starts at 10 unless another quota is selected.

| Bot team | `BotTeam` JSON value | Result |
| --- | --- | --- |
| Any | `"any"` | Default; restores original team cvars when leaving a forced side. |
| CT | `"ct"` | Bots play CT; humans play T. |
| T | `"t"` | Bots play T; humans play CT. |

Selecting CT or T moves humans off the bot side, kicks and re-adds bots, and restarts the round. Returning to Any restores `bot_join_team`, `mp_humanteam`, `mp_autoteambalance`, and `mp_limitteams`, then restarts without kicking bots.

Team selection survives map changes and remains active until returned to Any, restored, or cleared by a mode switch. RCT saves the active side and original values so they remain restorable after plugin reload. It can combine with gameplay modes, such as ScoutzKnives against 20 bots; Deathmatch and One in the Chamber reject team restrictions explicitly.

## Rollback and map changes

### Switching and ending modes

Universal and `!fun` share a pre-mode snapshot. Replacement restores the outgoing snapshot before capturing a fresh one for the next mode. Changed bot, physics, and other tracked cvars return to their captured values.

A server-side operation lock prevents overlapping supported-interface changes, including changes across Deathmatch map reloads. The lock expires after two minutes as a recovery safeguard. A named profile-disable command only succeeds while that profile owns the active state, preventing stale cleanup from affecting its replacement.

No Bots is independent: it reasserts quota zero after maps and restores its captured quota without restarting the round.

### Map carryover

- Modes started by `!fun` retain their temporary choices across map changes for the current run. Scheduled rotation retains its configured choices. Both reapply after map configs load without recapturing rollback values or adding another round restart. Temporary menu choices are not written into recovery settings; plugin reload/server restart recovers using configured defaults.
- Deathmatch, No Bots, and One in the Chamber retain their own dedicated reapply handlers.
- Super Hero resets map-specific state and reassigns powers to living players about five seconds after map start.
- Universal-owned modes use the app's map watcher for cvar reapplication, so **keep Universal open** for that part.

Stale callbacks cannot revive a replaced or rolled-back mode. Transition guards prevent teardown of player entities from being mistaken for an immediate empty-server rollback.

### Empty-server policy

| Top-level setting | Default | Behavior |
| --- | --- | --- |
| `RollbackWhenServerEmpty` | `true` | Automatically ends the active mode after sustained absence of humans. |
| `EmptyServerRollbackDelaySeconds` | `300` | Five-minute grace period; range `0`-`3600`. `0` removes the grace period. |

Bots and SourceTV do not count as humans. Returning or connecting humans cancel pending rollback. Map changes and repeated checks do not extend the original deadline; expiry rechecks the roster and transition state.

Explicit Rollback is immediate. Replacement and plugin unload cancel pending timers. Disabling the policy keeps modes active indefinitely on an empty server. Editing a pending delay retains the original departure time.

Use `css_rct_empty_server_config <on|off> <seconds>` to save the policy. Save failure leaves the old policy intact. Universal's **Empty-server rollback (server-wide)** panel reads and edits the same policy in minutes (`0`-`60`) without starting a mode.

`css_rct_status` reports the active profile/owner, **Map transition**, **Last mode end**, and **Empty-server grace**. Check these when diagnosing a mode that ended unexpectedly.

## Configuration and upgrades

No JSON editing is needed to try a mode with defaults. CounterStrikeSharp generates:

```text
game/csgo/addons/counterstrikesharp/configs/plugins/RCT/RCT.json
```

Each gameplay mode has an independent section. `!fun` and `css_rct_fun_apply` start from those defaults; Universal and scheduled tasks can supply their configured settings. Unload RCT before hand-editing, save valid JSON, then load it or restart the server. Roll back and restart an active mode to use changed defaults; active runs retain their starting settings.

New Fun Stuff settings have bots disabled. Saved choices are preserved. Weapon Rotation defaults to category rotation every five minutes, infinite ammunition, and category voting.

### Upgrading an existing `RCT.json`

RCT uses schema version `3` in `ConfigVersion`. Older schemas are backed up in `migration-backups` beside the configuration, then migrated while preserving saved settings, explicit empty values, and unknown properties. Unsupported newer schemas are rejected without overwriting the file. `css_rct_status` reports loaded and supported schema versions.

A schema migration is not the same as regenerating every optional field. Missing fields use their defaults; do not assume every new option will appear in an already-current file.

- **Keep your settings:** unload RCT and add the desired fields using the references below.
- **Regenerate all defaults:** unload RCT, back up and rename `RCT.json`, then load RCT. Reapply custom values to the new file.
- **Save Bot Control preferences:** **APPLY BOT SETTINGS** saves that panel's preferences. Gameplay wizards never save temporary choices, even when Bot Control or debug settings are saved during a run.

Existing 1.0.2 configurations may already contain `SuperHero`, `OneInTheChamber`, `BotControl`, `RollbackWhenServerEmpty`, and `EmptyServerRollbackDelaySeconds`. Keep their saved values unless you intend to change them.

Companion maintains `rollback-state.json` beside `RCT.json`. Do not edit it: it holds the active mode's pre-change cvars and shared rollback metadata, not credentials. Bot-team restore metadata is maintained automatically too.

### Hide entries from the in-game menu

Set top-level `"HiddenFunMenuModes": ["deathmatch", "nobots"]` to hide those entries from `!fun`. Use exact tokens from [Available modes](#available-modes); the default `[]` shows everything. Bot Control and Rollback always remain visible.

This is menu visibility, not access control: Universal, scheduled tasks, and direct commands can still start hidden modes. Reload the configuration/plugin after editing; open administrative menus close when visibility changes.

### Settings with fixed values (hand-editing `RCT.json`)

Menus show friendly labels, but JSON stores short internal values. Use **exactly** the values below (quoted and lowercase where shown). Labels such as `"Weapon categories"` or `"Team respawn"` are not valid stored values. Text is case-sensitive unless noted.

**`WeaponRotation`** — an invalid value prevents Weapon Rotation from starting and shows an error in chat/console.

| Setting | JSON values (menu label) | Default |
| --- | --- | --- |
| `RotationKind` | `"weapon"` (Individual weapons), `"category"` (Weapon categories) | `"category"` |
| `RotationTrigger` | `"time"` (Timed interval), `"round"` (Every round; requires `"Combat": "rounds"`) | `"time"` |
| `Distribution` | `"shared"` (Same weapon for everyone), `"individual"` (Independent per-player weapon decks) | `"shared"` |
| `Announcements` | `"popup"` (Popup), `"chat"` (Chat), `"both"` (Popup + chat) | `"both"` |
| `Combat` | `"rounds"` (Team rounds), `"teams"` (Team respawn), `"ffa"` (FFA respawn) | `"rounds"` |
| `AutoBalanceTeams` | `true` or `false`; team play only, fixed-team Bot Control is exempt | `true` |
| `Armor` | `"none"` (None), `"kevlar"` (Kevlar), `"helmet"` (Kevlar + helmet) | `"helmet"` |
| `BotQuotaMode` | `"normal"`, `"fill"`, `"match"` | `"fill"` |
| `SelectedCategories` | Any of `"Pistols"`, `"Shotguns"`, `"Machine guns"`, `"SMGs"`, `"Assault rifles"`, `"Sniper rifles"` (exact spelling, no duplicates, at least one) | all six |
| `SelectedWeapons` | Weapon IDs, no duplicates: Pistols `glock`, `hkp2000`, `usp_silencer`, `elite`, `p250`, `tec9`, `fiveseven`, `cz75a`, `deagle`, `revolver`; Shotguns `nova`, `mag7`, `sawedoff`, `xm1014`; Machine guns `m249`, `negev`; SMGs `mp5sd`, `p90`, `mp7`, `mac10`, `mp9`, `bizon`, `ump45`; Assault rifles `galilar`, `famas`, `ak47`, `m4a1` (M4A4), `m4a1_silencer` (M4A1-S), `aug`, `sg556`; Sniper rifles `ssg08`, `awp`, `g3sg1`, `scar20` | all 34 |
| `Grenades` | Any of `"hegrenade"`, `"flashbang"`, `"smokegrenade"`, `"decoy"`, `"molotov"`, `"incgrenade"` (no duplicates) | `[]` |
| `Shuffle`, `CategoryVoting`, `Knives`, `Buying`, `Pickups`, `InfiniteAmmo`, `BotsEnabled`, `MapChooserPoolSwap` | `true` or `false` (no quotes) | `true`, `true`, `true`, `false`, `false`, `true`, `false`, `false` |
| `IntervalSeconds` / `DurationMinutes` / `RespawnDelaySeconds` / `ChatReminderSeconds` | Numbers: 5-3600 / 0-1440 (0 = unlimited) / 0.25-10 / 0-300 (0 = no periodic chat) | 300 / 0 / 1 / 30 |
| `BotQuota` | `null` (keep current) or 0-64 | `null` |
| `PayloadVersion` / `MapChooserPoolName` | Must stay `1` / `"Weapon Rotation"` | — |

At least one selected weapon must belong to a selected category. `CategoryVoting` only has an effect with `"RotationKind": "category"`.

**`BotControl`** — saved preferences for the live Bot Control layer.

| Setting | JSON values (menu label) | Default |
| --- | --- | --- |
| `BotTeam` | `"any"` (Any), `"ct"` (CT, humans play T), `"t"` (T, humans play CT); case-insensitive, unrecognized values become `"any"` | `"any"` |
| `HumanTeam` | `"any"`, `"ct"`, `"t"` | `"any"` |
| `Halftime` | `null` (Keep current), `false` (Disable halftime), `true` (Enable halftime; not allowed for FFA modes) | `null` |
| `BotDifficulty` | `null` (Keep current) or `0` Easy, `1` Normal, `2` Hard, `3` Expert, `4` Custom, `5` Elite/Peak (clamped to 0-5) | `null` |
| `BotsOnlyOnTeam`, `BotsEnabled` | `true` or `false` | `false`, `true` |
| `BotQuotaMode` / `BotQuota` | `"normal"`, `"fill"`, `"match"` / `null` or 0-64 | `"fill"` / `null` |

Do not edit fields starting with `Applied` or `Rollback`, or `HasSavedSettings` and `BotTeamOriginals`; RCT maintains them for safe restoration. Other modes' quota values are described in [Bot settings](#bot-settings).

## Command reference

Except for the permission-protected in-game menu, administrative commands are restricted to the server console or RCON.

### Modes and configuration

| Command | Purpose |
| --- | --- |
| `css_rct_status` | Report version, active state, grace/transition diagnostics, and capabilities. |
| `css_rct_version` | Report the plugin version. |
| `css_rct_admin_status` | Diagnose connected administrators' authorization/permission state without account IDs. |
| `css_rct_debug <0\|1>` | Disable/enable optional RCT diagnostics and save the choice. |
| `css_alive [target]` | Report alive/dead state and health; omit the target to list connected players. |
| `css_fun` / `!fun` / `/fun` | Open the built-in menu; requires `@css/fun`. |
| `css_rct_fun_apply <mode>` | Start a complete mode using `RCT.json`, rolling back the current mode first. |
| `css_rct_mode_enable <mode>` | Activate an isolated profile; lower-level integration command. |
| `css_rct_mode_disable <mode>` | Disable the named profile only when it is active. |
| `css_rct_mode_rollback` | Roll back the active mode regardless of its control interface. |
| `css_rct_superhero_config` | Set the original six power toggles and strengths for the server session (13 arguments). |
| `css_rct_superhero_extra_config` | Set the eight additional toggles and strengths for the server session (21 arguments; sent by Universal). |
| `css_rct_superhero_moon_config <velocity>` | Set runtime Moon Jumper launch velocity (`300`-`1500`). |
| `css_rct_superhero_notice <seconds>` | Set runtime popup duration (`0`-`30`; `0` disables). |
| `css_rct_superhero_reminder <seconds>` | Set runtime power chat reminders (`0`-`300`; `0` disables). |
| `css_rct_empty_server_config <on\|off> <seconds>` | Persist the server-wide empty-server policy (`0`-`3600` seconds). |
| `css_rct_chamber_enable <on\|off> <respawn-seconds>` | Enable Chamber rules after its profile; the toggle controls Deagle-headshots-only. |

### Player commands

| Command | Purpose |
| --- | --- |
| `css_rct_give <target> <weapon>` | Give an item to alive targets. |
| `css_rct_strip <target>` | Remove all weapons. |
| `css_rct_respawn <target>` | Respawn targets. |
| `css_rct_health <target> <amount>` | Set health (`1`-`100000`). |
| `css_rct_speed <target> <multiplier>` | Set movement speed (`0.1`-`10`). |
| `css_rct_gravity <target> <scale>` | Set gravity (`0`-`10`). |
| `css_rct_freeze <target>` / `css_rct_unfreeze <target>` | Freeze targets or restore walking movement. |
| `css_rct_money <target> <amount>` | Set money (`0`-`65535`). |
| `css_rct_god <target> <on\|off>` | Toggle damage immunity. |
| `css_rct_noclip <target> <on\|off>` | Toggle noclip. |
| `css_rct_reset <target>` | Reset managed speed, gravity, movement, and damage state. |

Weapon names accept the `weapon_` prefix or its omission: `taser` and `weapon_taser` are equivalent.

### Low-level loadout and rule commands

These are integration building blocks, not complete mode starts.

| Commands | Purpose |
| --- | --- |
| `css_rct_zeus_enable` / `css_rct_zeus_disable` | Managed Zeus loadouts. |
| `css_rct_knife_enable` / `css_rct_knife_disable` | Managed team-knife loadouts. |
| `css_rct_pistols_enable` / `css_rct_pistols_disable` | Managed team-default pistol loadouts. |
| `css_rct_scoutzknives_enable` / `css_rct_scoutzknives_disable` | Managed SSG 08 and team-knife loadouts. |
| `css_rct_awp_loadout_enable` / `css_rct_awp_loadout_disable` | Managed AWP-only loadouts. |
| `css_rct_grenades_enable` / `css_rct_grenades_disable` | Managed HE-grenade loadouts. |
| `css_rct_fire_grenades_enable` / `css_rct_fire_grenades_disable` | Managed Molotov/incendiary loadouts. |
| `css_rct_loadout_disable` | Disable the active managed loadout. |
| `css_rct_buy_disable` / `css_rct_buy_enable` | Disable buying or restore normal buy-zone/purchase behavior. |
| `css_rct_deathmatch_enable` / `css_rct_deathmatch_disable` | Toggle a healthshot reward every three consecutive kills; reset streaks. |
| `css_rct_ground_clear_enable` / `css_rct_ground_clear_disable` | Toggle per-round removal of map-placed/dropped weapons, preserving C4. Map weapons return next round when disabled. |

## Player targets and examples

| Target | Matches |
| --- | --- |
| `@all` | All connected humans and bots |
| `@ct` / `@t` | Counter-Terrorists / Terrorists |
| `@spec` | Spectators |
| `@alive` / `@dead` | Alive / dead players |
| `@bot` / `@human` | Bots / humans |
| `#<userid>` | One server user ID, such as `#12` |
| `<SteamID64>` | One Steam account |
| `<exact name>` | One or more exact, case-insensitive name matches |

Quote names containing spaces:

```text
css_rct_give @all taser
css_rct_health @ct 250
css_rct_speed "Player Name" 1.5
css_rct_gravity #12 0.5
css_rct_god @human on
css_rct_reset @human
```

## Managed player state

Speed, gravity, freeze, god mode, and noclip are tracked by SteamID and reapplied after spawn. `css_rct_reset` restores normal values; unloading restores managed movement and damage values.

Managed inventories are applied to living humans and bots immediately, after later spawns, and on human bot takeover. RCT uses CounterStrikeSharp's inventory API and preserves the Terrorist bomb carrier's C4 where the mode allows it. Disabling a managed loadout does not strip players again; normal inventories return through usual game spawn/round behavior.

In grenade modes, humans retain a team knife as CS2's post-throw fallback. RCT restores grenade ammo and selects HE slot 6 or fire-grenade slot 10 after throws. Bots receive no knife so their AI cannot prefer melee. On takeover, inventory operations use the live bot controller and slot-selection commands use the human controller.

While buying is disabled, RCT removes buy-zone access each tick without cancelling normal weapon selection or C4 planting. Chamber's separate rules disable C4 objectives.

RCT does not read, execute, or modify `server.cfg` or other server configuration scripts.

## Troubleshooting and validation

Run `css_rct_version` and `css_rct_status` from the server console or RCON first to confirm the loaded build and active settings.

| Symptom | Check |
| --- | --- |
| RCT appears as `Unknown (Unknown)` | Install the complete compatible CounterStrikeSharp runtime and current RCT release, remove the older DLL, then restart instead of hot-reloading across the framework change. |
| `!fun` is unavailable | Confirm RCT loaded and the administrator has `@css/fun`. No external menu plugin is needed. |
| A new option is missing from JSON | Existing files are not regenerated on upgrade; follow the configuration upgrade steps above. |
| Settings Apply cannot save | Check write access to `configs/plugins/RCT/RCT.json` and the reported error path. Restore ordinary permissions after testing. |
| Super Hero assigns a different power | Disable other powers when testing one; assignments are balanced across enabled powers, not selected manually per player. |
| A mode ends after everyone leaves | Check the empty-server policy, grace timer, and last mode-end reason with `css_rct_status`. |
| Universal-owned cvars do not carry over | Keep the desktop app open for its map watcher; inspect active profile/owner and transition diagnostics. |
| Administrator access disappears after a map change | Run `css_rct_admin_status` while the affected player is connected. It distinguishes pending Steam authorization from missing administrator permission without printing account IDs. Reloading the admin file does not authorize a player. |
| Changed defaults do not affect a running mode | Roll back and start again; active settings are preserved through plugin reloads. |
| Category voting is missing | Use category rotation, enable CategoryVoting, and select weapons in at least two categories. Very short intervals can skip voting. Use `!wrvote` to reopen an active ballot. |
