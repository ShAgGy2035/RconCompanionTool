# RCON Tool Companion

**Version:** 1.0.2 | **Author:** ShAgGy

RCON Tool Companion (RCT) is a standalone CounterStrikeSharp plugin for CS2 Fun Stuff modes and player-level commands. It includes its own in-game administration menu and does not depend on SimpleAdmin or a separate menu plugin.

The [Ultimate CS2 RCON/Server Tool](https://github.com/ShAgGy2035/Ultimate-RCON-Server-Tool) is optional. Use RCT alone with `!fun`, or use the desktop tool for mode settings, scheduled rotations, RCON commands, and server management.

See the [1.0.2 release notes](https://github.com/ShAgGy2035/RconCompanionTool/releases/tag/v1.0.2) for release history.

## Contents

- [Requirements](#requirements)
- [Install and verify](#install-and-verify)
- [Start a mode](#start-a-mode)
- [Available modes](#available-modes)
- [Super Hero: all fourteen powers](#super-hero-all-fourteen-powers)
- [One in the Chamber](#one-in-the-chamber)
- [Bot settings](#bot-settings)
- [Configuration and upgrades](#configuration-and-upgrades)
- [Rollback and map changes](#rollback-and-map-changes)
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

1. Extract `RconToolCompanion-v1.0.2.zip` into the server's `game/csgo/` directory. The archive starts at `addons/`.
2. Confirm the deployed plugin is at:

   ```text
   game/csgo/addons/counterstrikesharp/plugins/RCT/RCT.dll
   ```

3. Restart the server or load RCT through CounterStrikeSharp.
4. Run `css_rct_status` from the server console or RCON. A successful response starts with `[RconToolCompanion] Ready.` and reports the version and capabilities.
5. Grant trusted in-game administrators the CounterStrikeSharp permission `@css/fun`.

The short `RCT` directory is intentional: affected Linux container filesystems can fail to canonicalize a longer plugin path. Keep the deployed directory and DLL named `RCT` and `RCT.dll`.

## Start a mode

### In game

Type `!fun` or `/fun` in chat with the `@css/fun` permission.

| Key | Action |
| --- | --- |
| W / S | Move through options |
| E | Select |
| A | Return to the previous menu |
| R | Exit |

The menu holds the player in place while open. Select a gameplay mode, review its bot options, and choose **Start**. Bot edits in a mode submenu are saved to `RCT.json` and used when that mode starts. **No Bots** starts directly; the separate **Bot control** panel uses **Apply now**.

Choose **Rollback** to end the active mode and restore captured server settings. In-game administrators do not need an RCON password or the desktop app.

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

## Available modes

Use the token below with `css_rct_fun_apply`. In `!fun`, No Bots and Bot control appear first, gameplay modes follow alphabetically, and Rollback is last.

| Mode | Token | Overview |
| --- | --- | --- |
| No Bots | `nobots` | Sets bot quota to zero and removes bots without changing weapons, buying, or ground cleanup. |
| AWP / Sniper Wars | `awp` | Managed AWP-only loadouts; configurable forced loadout and infinite ammo. |
| Bhop | `bhop` | Bunnyhopping and optional stamina-limit removal, with unrestricted weapons. |
| Deathmatch | `deathmatch` | Configurable respawns, random spawns, duration, ammo, bot AI, and kill-streak medikit rewards. |
| Grenade Wars | `grenades` | Managed HE grenades with replenishment after throws. |
| Headshot Only | `headshot` | Headshot rules with unrestricted weapons. |
| Knife Arena | `knife` | Managed team-knife loadouts. |
| Molotov / Incendiary Wars | `fire-grenades` | Managed team-appropriate fire grenades with replenishment. |
| One in the Chamber | `oneinthechamber` | Native free-for-all with a Deagle, knife, and one starting bullet. |
| Pistols Only | `pistols` | Managed team-default pistol loadouts, with optional headshots-only rules. |
| ScoutzKnives | `scoutzknives` | SSG 08 and team knife, with configurable movement and physics. |
| Super Hero | `superhero` | One of fourteen enabled powers per life; normal weapons and buying. |
| Surf | `surf` | Configurable surf physics, stamina, round time, and freeze time. |
| Zeus Wars | `zeus` | Managed Zeus-only loadouts. |

**Bot Control** is a separate panel, not a replacement gameplay mode. It can adjust bots while another mode remains active.

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

| Setting | Default | Behavior |
| --- | --- | --- |
| `BotsEnabled` | `true` | When disabled, sets quota to zero and kicks current bots; quota and quota mode are ignored. |
| `BotQuota` | `null` | Preserves the current server quota. Set `1`-`64` to enforce a count. |
| `BotQuotaMode` | `fill` | Accepts `normal`, `fill`, or `match`. `fill` replaces bots as humans join to maintain the configured total quota. |

All quota modes, including standalone Bot Control, default to `fill`. Existing explicit `normal` or `match` selections are preserved. In a mode's `!fun` submenu, administrators can select the current quota or presets from 1 to 20.

### Standalone Bot Control

Changes in **Bot control** are staged until **Apply now**. Apply saves preferences to `RCT.json` before changing the live server; a save failure leaves the server unchanged. Enabling bots with a zero current quota starts at 10 unless another quota is selected.

| Bot team | Result |
| --- | --- |
| Any | Default; restores original team cvars when leaving a forced side. |
| CT | Bots play CT; humans play T. |
| T | Bots play T; humans play CT. |

Selecting CT or T moves humans off the bot side, kicks and re-adds bots, and restarts the round. Returning to Any restores `bot_join_team`, `mp_humanteam`, `mp_autoteambalance`, and `mp_limitteams`, then restarts without kicking bots.

Team selection survives map changes and remains active until returned to Any. RCT saves the active side and original values so they remain restorable after plugin reload. It can combine with gameplay modes, such as ScoutzKnives against 20 bots; Deathmatch ignores team assignment.

## Configuration and upgrades

On first load, CounterStrikeSharp generates:

```text
game/csgo/addons/counterstrikesharp/configs/plugins/RCT/RCT.json
```

Each mode has its own section. `!fun` and scheduled rotations use these defaults; Universal can supply its exposed settings. To edit manually, unload RCT, edit valid JSON, then load RCT or restart the server. Numeric settings are clamped to their supported ranges.

ScoutzKnives includes air acceleration, gravity, bunnyhopping, automatic bunnyhopping, fall-damage scale, and friction. Surf includes cheats, acceleration, air acceleration, gravity, both bunnyhop switches, jump/landing stamina costs, round time, and freeze time. Grenade Wars, fire-grenade modes, and Zeus Wars always enforce their required loadouts and do not expose a `ManagedLoadout` option.

### Upgrading an existing `RCT.json`

An upgrade does not automatically insert new options into an existing file. **Missing fields use their defaults**; saved values remain in effect.

- **Keep your settings:** unload RCT and add the desired fields manually.
- **Regenerate all defaults:** unload RCT, back up and rename `RCT.json`, then load RCT. Reapply custom values to the new file.
- **Save from `!fun`:** a mode-submenu bot edit saves the complete current configuration, adding missing fields with their current values. Bot Control does the same when **Apply now** is selected.

Version 1.0.2 adds the `SuperHero`, `OneInTheChamber`, and `BotControl` sections, plus the top-level `RollbackWhenServerEmpty` and `EmptyServerRollbackDelaySeconds` settings. All Super Hero fields are listed above.

Companion maintains `rollback-state.json` beside `RCT.json`. Do not edit it: it holds the active mode's pre-change cvars and shared rollback metadata, not credentials. Bot-team restore metadata is also maintained automatically.

## Rollback and map changes

### Switching and ending modes

Universal and `!fun` share a pre-mode snapshot. Replacement restores the outgoing snapshot before capturing a fresh one for the next mode. Changed bot, physics, and other tracked cvars return to their captured values.

A server-side operation lock prevents overlapping supported-interface changes, including changes across Deathmatch map reloads. The lock expires after two minutes as a recovery safeguard. A named profile-disable command only succeeds while that profile owns the active state, preventing stale cleanup from affecting its replacement.

No Bots is independent: it reasserts quota zero after maps and restores its captured quota without restarting the round.

### Map carryover

- Modes started by `!fun` or scheduled rotation replay saved active commands after map configs load, without recapturing rollback values or adding another round restart.
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

## Command reference

Except for the permission-protected in-game menu, administrative commands are restricted to the server console or RCON.

### Modes and configuration

| Command | Purpose |
| --- | --- |
| `css_rct_status` | Report version, active state, grace/transition diagnostics, and capabilities. |
| `css_rct_version` | Report the plugin version. |
| `css_fun` / `!fun` / `/fun` | Open the built-in menu; requires `@css/fun`. |
| `css_rct_fun_apply <mode>` | Start a complete mode using `RCT.json`, rolling back the current mode first. |
| `css_rct_mode_enable <mode>` | Activate an isolated profile; lower-level integration command. |
| `css_rct_mode_disable <mode>` | Disable the named profile only when it is active. |
| `css_rct_mode_rollback` | Roll back the active mode regardless of its control interface. |
| `css_rct_superhero_config` | Set the original six power toggles and strengths for the server session (13 arguments). |
| `css_rct_superhero_extra_config` | Set the eight additional toggles and strengths for the server session (21 arguments; sent by Universal). |
| `css_rct_superhero_moon_config <velocity>` | Set runtime Moon Jumper launch velocity (`300`-`1500`). |
| `css_rct_superhero_notice <seconds>` | Set runtime popup duration (`0`-`30`; `0` disables). |
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

| Symptom | Check |
| --- | --- |
| RCT appears as `Unknown (Unknown)` | Install the complete compatible CounterStrikeSharp runtime and current RCT release, remove the older DLL, then restart instead of hot-reloading across the framework change. |
| `!fun` is unavailable | Confirm RCT loaded and the administrator has `@css/fun`. No external menu plugin is needed. |
| A new option is missing from JSON | Existing files are not regenerated on upgrade; follow the configuration upgrade steps above. |
| Settings Apply cannot save | Check write access to `configs/plugins/RCT/RCT.json` and the reported error path. Restore ordinary permissions after testing. |
| Super Hero assigns a different power | Disable other powers when testing one; assignments are balanced across enabled powers, not selected manually per player. |
| A mode ends after everyone leaves | Check the empty-server policy, grace timer, and last mode-end reason with `css_rct_status`. |
| Universal-owned cvars do not carry over | Keep the desktop app open for its map watcher; inspect active profile/owner and transition diagnostics. |

Earlier builds saved through a framework helper that derived `RconToolCompanion/RconToolCompanion.json` from the assembly name. Current bot, bot-team, and empty-server saves explicitly use `RCT/RCT.json`; changing permissions on that file could not fix the earlier wrong-path issue. RCT does not migrate or overwrite the long-name configuration.

Useful capability tags include `superhero-expanded`, `superhero-notice`, `nobots`, `one-in-the-chamber`, `chamber-native-ffa`, `mode-map-reapply`, `map-transition-guard`, `empty-server-grace`, and `empty-server-settings`. Universal uses capabilities to reject unsupported older builds.

Release builds and focused regression checks cover power allocation, damage/jump predicates, ammo accounting, map/rollback guards, grace deadlines, popup timing/identity, and configuration persistence. Hosted live tests have confirmed settings saving, Super Hero map-rotation carryover, readable spectator notices, and moving Chamber bots.

These checks do **not** establish full gameplay validation. Remaining live checks include broader Super Hero power behavior and reload animations, Chamber HUD/scoring and transitions, actual empty-server expiry/reconnect scenarios, exact popup duration, and residual HUD dimming. No claim of flicker-free rendering is made.
