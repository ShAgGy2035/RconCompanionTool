# RCON Tool Companion

Standalone CounterStrikeSharp plugin with optional integration into [ShAgGy's Ultimate CS2 RCON/Server Tool](https://github.com/ShAgGy2035/Ultimate-RCON-Server-Tool).

Version: 1.0.2  
Author: ShAgGy

Release history: [v1.0.2 release notes](Release%20Notes-1.0.2.md).

RCON Tool Companion provides player-level fun commands that CS2's native RCON console cannot perform reliably. It is independent of SimpleAdmin. Administrative console commands are restricted to the server console or RCON; the in-game `!fun` / `/fun` menu requires the `@css/fun` permission.

The desktop RCON/Server Tool is **optional**: trusted administrators can use `!fun` to select modes, adjust available bot settings, and roll back modes without installing or running the desktop app. Companion runs on the CS2 server and uses its own `RCT.json` configuration. The [desktop tool](https://github.com/ShAgGy2035/Ultimate-RCON-Server-Tool) adds a convenient UI for server/RCON management, exposed mode settings, plugin maintenance, and scheduling; it is not a prerequisite for in-game Fun Stuff.

## Requirements

- A Counter-Strike 2 dedicated server
- Metamod:Source
- Full CounterStrikeSharp 1.0.376 runtime (or a compatible newer runtime). The expanded powers use its native pre-damage listeners and hull traces; an older native runtime is not supported simply by replacing its API DLL.
- No menu plugin is required; `!fun` uses Companion's built-in WASD menu

## Installation

1. Extract the release ZIP into the CS2 server's `game/csgo/` directory. The archive starts at `addons/`, not `game/csgo/`.
2. Confirm this file exists:
	`game/csgo/addons/counterstrikesharp/plugins/RCT/RCT.dll`
3. Restart the server or load the plugin through CounterStrikeSharp.
4. Run `css_rct_status` through RCON. A successful response starts with `[RconToolCompanion] Ready.`

This release targets .NET 10 and CounterStrikeSharp API 1.0.376. The project builds successfully against that API. If `css_plugins list` shows `Unknown (Unknown)`, remove the older RCT DLL, install this complete release, and restart the server instead of hot-reloading across the framework change.

`!fun` uses a built-in center-screen WASD menu: **W/S** move, **E** selects, **A** returns to the previous menu, and **R** exits. The player is held in place while the menu is open. Companion 1.0.2 no longer depends on MenuManagerAPI, MenuManager, or PlayerSettingsCS2; they can stay installed for other plugins but are not used by RCT.

The short `RCT` plugin directory is intentional. On affected Linux container filesystems, the .NET host cannot canonicalize a longer plugin path even though CounterStrikeSharp can discover it. The DLL retains the full product name for clarity.

On first load, CounterStrikeSharp generates the standalone mode configuration at:

`game/csgo/addons/counterstrikesharp/configs/plugins/RCT/RCT.json`

In-game and RCON settings saves explicitly target this deployed `RCT/RCT.json` path. Earlier builds used a CounterStrikeSharp helper that derived `RconToolCompanion/RconToolCompanion.json` from the assembly name instead; changing permissions on `RCT.json` could not fix a failure writing that different path. The corrected build uses the same RCT path for bot settings, bot-team restore state, and empty-server policy. It does not migrate or overwrite files under the long-name path. Restore ordinary permissions after testing; real filesystem write restrictions still apply, and save errors now report the intended RCT path. Local file creation/update/roundtrip checks passed, and the user has confirmed hosted settings Apply and JSON saving now work.

Its mode sections mirror Universal's Fun Stuff options. ScoutzKnives exposes air acceleration, gravity, bunnyhopping, automatic bunnyhopping, fall-damage scale, and friction. Surf exposes cheats, acceleration, air acceleration, gravity, both bunnyhop switches, jump and landing stamina costs, round time, and freeze time. Grenade Wars, Fire Grenade Wars, and Zeus Wars always enforce their required managed loadouts and no longer expose a `ManagedLoadout` setting. Every mode also has `BotsEnabled`, `BotQuota`, and `BotQuotaMode`. `BotsEnabled` defaults to `true`; `BotQuota` defaults to `null`, which preserves the server's current quota. Set `BotQuota` from 1 to 64 to enforce an exact count for that mode. `BotQuotaMode` accepts `normal`, `fill`, or `match` and defaults to `normal`; `fill` replaces bots as human players join to maintain the configured total quota. When bots are disabled, RCT ignores the quota and quota mode, sets `bot_quota 0`, and kicks current bots. Rollback or a mode switch restores every changed cvar, including bot and physics values, to its captured pre-mode value. Edit the JSON while the plugin is unloaded, then load or restart RCT. Invalid values are clamped to supported ranges.

Administrators with `@css/fun` can also change each mode's `BotsEnabled`, `BotQuota` (current or a preset from 1 to 20), and `BotQuotaMode` in game. Selecting any mode except No Bots in `!fun` opens that mode's submenu with **Start** (showing the bot settings it will use) followed by the bot settings. **A** returns to the previous menu. Changes are saved to `RCT.json` and are used when the mode is started.

The standalone `!fun` **Bot control** panel is staged: changing bot enablement, quota, quota mode, or team only edits the open draft. Choose **Apply now** to save the preferences to `RCT.json` and apply them to the live server. If saving fails, no server changes are applied. Enabling bots with a zero current quota starts with a quota of 10 unless another quota is selected.

`RollbackWhenServerEmpty` (default `true`) rolls back the active Fun Stuff mode, whether applied by Universal or `!fun`, only after the server stays empty for `EmptyServerRollbackDelaySeconds` (default `300`, five minutes). Humans returning, including connecting/reconnecting players, cancel the pending rollback. Bots and SourceTV do not count. The deadline is retained across maps and repeated empty checks do not extend it. At expiry, Companion rechecks the roster and map-transition state before rolling back. Explicit **Rollback** is still immediate and cancels pending grace timers; mode replacement and plugin unload also cancel them. Set `RollbackWhenServerEmpty` to `false` to keep modes indefinitely on an empty server. The delay accepts `0`-`3600` seconds; `0` removes the grace period. `css_rct_status` shows **Empty-server grace** with remaining time, and the updated build advertises `empty-server-grace`. Builds and focused deadline/cancellation checks passed; live transition behavior still needs testing.

**Super Hero** (`css_rct_mode_enable superhero`, also in `!fun`) keeps normal weapons and buying and assigns one of fourteen enabled powers on each spawn. The original powers are Speedster (`SpeedMultiplier`, default 1.6x), Moon Jumper (`LowGravityScale`, default 0.4), Tank (`TankHealth`, default 250, plus full armor), Ghost (`GhostVisibilityPercent`, default 15), Vampire (heals `VampirePercent`, default 50%, of damage dealt), and Regenerator (heals `KillHealAmount`, default 50, per enemy kill). Vampire and Regenerator healing stops at `MaxHealth` (default 200). Universal exposes all fourteen toggles and their strengths directly in its Super Hero panel; `!fun` uses the `SuperHero` section of `RCT.json`. Rollback removes all powers.

Super Hero also shows a non-interactive center-screen notice with the assigned power name and the same description as chat. `SuperHero.PowerNoticeSeconds` defaults to `5`; set it from `0` to `30`, with `0` disabling player and spectator popups. Universal exposes **Power popup (seconds)**. Chat remains available as a fallback. Spectators see the power of the living player or bot they are watching, including the subject's name and a bot label; switching targets or a new power/life starts a fresh notice. A viewer's `!fun` menu takes priority and pauses an existing notice's clock. Popups do not freeze movement or accept menu input, and are cleared on expiration, mode exit, or map reset. Focused clock, target-identity, and safe HTML checks passed; actual in-game spectator/HUD presentation still needs testing. Updated Companion reports `superhero-notice` capability.

Power notices maintain cached HTML on each server tick, matching the existing `!fun` menu's display cadence, rather than relying on one HUD event to remain visible. Target resolution and HTML generation are still cached/throttled; the configured server-side clock controls expiry and menu pause/resume. The single-event revision was reported to disappear after about one second, so its native duration was not sufficient on the tested client. Spectator switching has been reported working; the new per-tick presentation still needs a live duration/pulsing/readability retest.

The eight additional powers default to enabled. Their boolean JSON keys match the power names without spaces:

| Power / Key | Effect | Strength Keys And Defaults |
| --- | --- | --- |
| `Juggernaut` | Less incoming damage, slower movement | `JuggernautDamageReductionPercent`: 35; `JuggernautSpeedMultiplier`: 0.75 |
| `SecondWind` | Cancels one lethal incoming hit per life and restores survival health | `SecondWindHealth`: 35 |
| `AmmoRecycler` | Enemy kills refill the active magazine | No numeric setting |
| `Berserker` | Increased damage and speed while below the health threshold | `BerserkerHealthThreshold`: 35; `BerserkerDamageMultiplier`: 1.5; `BerserkerSpeedMultiplier`: 1.3 |
| `Medic` | Heals nearby living teammates with line of sight, capped at 100 health | `MedicHealAmount`: 5; `MedicRadius`: 300 units; `MedicIntervalSeconds`: 2 |
| `Blink` | Use key (normally E) attempts a forward teleport; blocked paths, occupied destinations, and unsupported drops are rejected | `BlinkDistance`: 250 units; `BlinkCooldownSeconds`: 8 |
| `DoubleJump` | Release Jump and press again in the air for one extra jump, reset on landing | `DoubleJumpVelocity`: 250 |
| `Gunslinger` | Shortened reload completion, transferring existing reserve ammo rather than creating ammo | `GunslingerReloadMultiplier`: 0.5 (half the normal remaining reload time) |

Bots trigger Blink automatically when an enemy is ahead and automatically use Double Jump while descending. Blink is suppressed while the Fun Stuff menu is open; selecting a menu option does not activate it. Reload completion is cancelled when the weapon, life, power, or map changes. Compile and focused logic checks passed; live damage, tracing, jump, healing, and reload animation behavior still needs server testing. Universal requires the `superhero-expanded` capability for this panel.

**One in the Chamber** (`oneinthechamber` token) is an isolated native FFA mode with Companion-controlled respawning in Universal, `!fun`, and scheduled rotations. Each spawn grants the stock Desert Eagle, team knife, exactly one bullet, and no reserve ammo. Each non-self player kill, including knife kills and kills of players on the same nominal team, earns one bullet. Missing your shot leaves the knife as the way to earn another; earned bullets can accumulate up to 64 total. Buying, other weapons, map-placed weapons, and C4 objectives are disabled. A per-life ammo ledger prevents reloads, pickups, or bot takeovers from creating extra bullets.

The `OneInTheChamber` section in `RCT.json` supports `HeadshotsOnly` (default `false`), `RespawnDelaySeconds` (default `1`, range `0.25`-`10`), and the standard `BotsEnabled`, `BotQuota`, and `BotQuotaMode` fields. By default Deagle and knife hits are lethal. With Deagle headshots-only enabled, Deagle body hits do no damage while knife hits remain lethal. Universal exposes both rule options; `!fun` and rotations use the config defaults. Chamber owns its native Deathmatch FFA rules independently of the Deathmatch mode: `game_type 1`, `game_mode 2`, and `mp_dm_teammode 0` select individual scoring, with random spawns and Companion-controlled respawns. Chamber clears `bot_stop`, forces the stock bot behavior tree (`mp_bot_ai_bt ""`), and allows bots and humans on either team; these settings are reapplied after map loads and at round prestart. The user has confirmed that bots move in Chamber. Entry and rollback reload the current map. Its guarded five-second map reapply adds no round restart; rollback disables its callbacks, restores captured cvars after map configs load, and only then starts a pending replacement mode. CT/T assignments may remain internally; they do not enable team scoring. Requires `one-in-the-chamber`, `chamber-native-ffa`, and full CSS 1.0.376 runtime. Roll back an active older Chamber session before upgrading. Native HUD/scoring and other broader gameplay/map-transition cases still require live testing.

**Bot team** in Universal's **Bot Control** option or `!fun` > Bot control places every bot on one side: **Any** (default), **CT** (humans play T), or **T** (humans play CT). Choosing CT or T moves humans off the bot side, kicks and re-adds bots, and restarts the round. Returning to **Any** restores the original `bot_join_team`, `mp_humanteam`, `mp_autoteambalance`, and `mp_limitteams` values and restarts without kicking bots. Original team values and the active team are saved by Companion so they remain restorable after a plugin reload. The setting combines with any mode (for example ScoutzKnives with 20 bots against one player), is re-applied after map changes, and stays active until set back to Any. Deathmatch ignores team assignment.

All bot quota modes default to `fill`, including standalone Bot Control. An explicitly saved `normal` or `match` selection remains unchanged.

Super Hero assigns from the least-used enabled powers among living teammates, independently for each team. With at least five powers enabled, a five-player team gets five different powers; repeats are balanced when the team outnumbers the enabled powers. Respawning players receive a different power when another equally suitable choice exists. Humans and bots have individual assignments.

Moon Jumper combines reduced gravity with a high launch velocity applied on the first airborne tick, within a one-second pending jump window. Hold Jump as you take off to queue the boost. `MoonJumperJumpVelocity` sets the upward velocity (default `1200` units/s; range `300`-`1500`); `LowGravityScale` multiplies gravity (default `0.275`, range `0.1`-`1`, where `1` is normal). The default approximates ScoutzKnives' `sv_gravity 220` on a standard 800-gravity server. Universal exposes both settings; the assignment message reports both. Enable only Moon Jumper when testing because each spawn otherwise receives a random enabled power. Existing `RCT.json` values are preserved on upgrade; adjust `LowGravityScale` and `MoonJumperJumpVelocity` there if `!fun` should use the new defaults.

### Map Changes

Universal's **Fun Stuff > Empty-server rollback (server-wide)** panel can read and save `RollbackWhenServerEmpty` and `EmptyServerRollbackDelaySeconds` without starting or replacing a mode. It displays the grace period in minutes (default `5`, range `0`-`60`); Companion stores seconds in `RCT.json`. The app loads the selected server's values before allowing **Apply settings**. `css_rct_empty_server_config <on|off> <seconds>` validates and saves the policy before changing runtime behavior; save failure leaves the old policy intact. Disabling automatic rollback cancels a pending grace timer; editing the delay preserves its original departure time. Requires the `empty-server-settings` capability. This server-wide policy also governs `!fun` and scheduled rotations. Focused contract/timing checks passed; live UI/RCON persistence testing is pending.

Active modes are intended to carry over between maps. `!fun` and scheduled-rotation modes now replay their saved active commands after map configs load and reassert managed loadouts without recapturing rollback values or sending another round restart. Deathmatch and No Bots retain their existing dedicated reapply timing; One in the Chamber retains its own guarded map handler. Super Hero clears map-specific power/life/cooldown state and reassigns powers to living players about five seconds after map start, for either control interface. Stale callbacks cannot revive a replaced or rolled-back mode. Builds and focused callback checks passed; live map-transition testing is still pending.

Universal-owned modes use the app's map watcher for cvar reapplication, so the app must remain open for that part. `RollbackWhenServerEmpty` still intentionally disables a mode when the last human leaves; map recovery does not override that rollback. Check `css_rct_status` for the active profile/owner and the log for an empty-server rollback if a mode disappears. The updated build reports `mode-map-reapply` in its capabilities.

The `map-transition-guard` prevents rollback while a known transition is tearing down player entities. With `empty-server-grace`, last-human departure instead begins the default five-minute grace period; map loading and roster checks do not shorten or repeatedly restart it. The five-second map-initialization callback can recheck the roster but cannot bypass the grace deadline. A genuinely empty server eventually rolls back after that deadline; explicit Rollback remains immediate. Status includes **Map transition**, **Last mode end**, and **Empty-server grace**; logs record the reason and transition state. The reported live loss of Super Hero still requires a server retest.

### Upgrading an existing `RCT.json`

CounterStrikeSharp only generates `RCT.json` when it does not exist, so upgrading Companion does not add new options to an existing file. You do not need to regenerate it: any option missing from the file uses its documented default. To see or change a new option, either:

- **Add it manually** (keeps your settings): unload RCT, add the option to `RCT.json`, then load or restart RCT. For 1.0.2, add `"RollbackWhenServerEmpty": true` at the top level, next to the mode sections.
- **Regenerate the file** (resets everything to defaults): unload RCT, delete or rename `RCT.json`, then load or restart RCT so CounterStrikeSharp writes a new file with every option. Re-apply any customized values afterwards.

Changing a bot setting from an in-game `!fun` mode submenu also saves the complete current configuration back to `RCT.json`, which adds any missing options with their current values. The standalone Bot Control panel adds a top-level `BotControl` section when **Apply now** saves and applies the settings.

| Version | New `RCT.json` options |
| --- | --- |
| 1.0.2 | `RollbackWhenServerEmpty` (top level, default `true`); `EmptyServerRollbackDelaySeconds` (top level, default `300`, range `0`-`3600`); `SuperHero` mode section, including eight additional enabled-by-default toggles, strength keys, and `PowerNoticeSeconds` (default `5`, `0` disables, max `30`); `BotControl` (top-level saved preferences; active bot-team restore state is maintained automatically); `OneInTheChamber` mode section (`HeadshotsOnly`, `RespawnDelaySeconds`, standard bot settings; `BotQuotaMode` defaults to `fill`) |

Companion automatically maintains `rollback-state.json` in the same `configs/plugins/RCT` directory. Do not edit this runtime file. It stores the active mode's pre-change ConVar commands so either Universal or `!fun` can roll back a mode applied by the other interface; it contains no credentials.

## Commands

| Command | Description |
| --- | --- |
| `css_rct_status` | Report version, Zeus state, and capabilities |
| `css_rct_version` | Report the plugin version |
| `css_fun` / `!fun` | Open the Fun Stuff menu; requires `@css/fun` |
| `css_rct_mode_enable <mode>` | Activate one isolated Fun Stuff mode profile |
| `css_rct_mode_disable <mode>` | Disable the named profile only when it is active |
| `css_rct_mode_rollback` | Roll back the active Fun Stuff mode regardless of which interface applied it |
| `css_rct_fun_apply <mode>` | Start a Fun Stuff mode with its `RCT.json` settings, rolling back the current mode first (used by Universal's scheduled Fun Stuff rotation) |
| `css_rct_superhero_config` | Set the original six powers and strengths for this server session (13 arguments) |
| `css_rct_superhero_extra_config` | Set the eight additional powers and strengths for this server session (21 arguments; sent by Universal) |
| `css_rct_superhero_notice <seconds>` | Set the runtime power-popup duration, from 0 to 30 seconds (`0` disables) |
| `css_rct_empty_server_config <on\|off> <seconds>` | Save server-wide empty-server rollback policy to `RCT.json`; grace range is 0-3600 seconds |
| `css_rct_chamber_enable <on\|off> <respawn-seconds>` | Activate One in the Chamber rules after its mode profile; on/off controls Deagle-headshots-only |
| `css_rct_give <target> <weapon>` | Give a weapon or item to alive targets |
| `css_rct_strip <target>` | Remove all weapons from targets |
| `css_rct_respawn <target>` | Respawn targets |
| `css_rct_health <target> <amount>` | Set health from 1 to 100000 |
| `css_rct_speed <target> <multiplier>` | Set movement speed from 0.1 to 10 |
| `css_rct_gravity <target> <scale>` | Set gravity from 0 to 10 |
| `css_rct_freeze <target>` | Freeze targets |
| `css_rct_unfreeze <target>` | Return targets to normal walking movement |
| `css_rct_money <target> <amount>` | Set money from 0 to 65535 |
| `css_rct_god <target> <on\|off>` | Enable or disable damage immunity |
| `css_rct_noclip <target> <on\|off>` | Enable or disable noclip |
| `css_rct_reset <target>` | Reset speed, gravity, movement, and damage state |
| `css_rct_zeus_enable` | Enable managed Zeus loadouts now and on spawn |
| `css_rct_knife_enable` | Enable managed knife-only loadouts now and on spawn |
| `css_rct_knife_disable` | Disable the managed knife-only loadout when active |
| `css_rct_pistols_enable` | Enable managed team-pistol-only loadouts now and on spawn |
| `css_rct_pistols_disable` | Disable the managed pistol loadout when active |
| `css_rct_scoutzknives_enable` | Enable managed SSG 08 and team-knife loadouts now and on spawn |
| `css_rct_scoutzknives_disable` | Disable the managed ScoutzKnives loadout when active |
| `css_rct_awp_loadout_enable` | Enable managed AWP-only loadouts now and on spawn |
| `css_rct_awp_loadout_disable` | Disable the managed AWP loadout when active |
| `css_rct_grenades_enable` | Enable managed HE-grenade-only loadouts now and on spawn |
| `css_rct_grenades_disable` | Disable the managed HE-grenade loadout when active |
| `css_rct_fire_grenades_enable` | Enable managed Molotov/incendiary-only loadouts now and on spawn |
| `css_rct_fire_grenades_disable` | Disable the managed fire-grenade loadout when active |
| `css_rct_loadout_disable` | Disable the active managed weapon loadout |
| `css_rct_zeus_disable` | Disable the managed Zeus loadout when active |
| `css_rct_buy_disable` | Close buy menus, remove buy-zone access, and block purchases |
| `css_rct_buy_enable` | Restore normal buy-zone and purchase behavior |
| `css_rct_deathmatch_enable` | Give a medikit (`weapon_healthshot`) every 3 consecutive kills; resets streaks |
| `css_rct_deathmatch_disable` | Disable Deathmatch kill-streak medikit rewards; resets streaks |
| `css_rct_ground_clear_enable` | Delete map-placed and dropped weapons left on the ground, every round (preserves the C4) |
| `css_rct_ground_clear_disable` | Stop removing ground weapons; map-placed weapons return next round |

Weapon names may be supplied with or without the `weapon_` prefix. For example, `taser` and `weapon_taser` are equivalent.

## In-Game Fun Menu

Grant trusted administrators the custom CounterStrikeSharp permission `@css/fun`. Those administrators can type `!fun` or `/fun` in chat and select any Fun Stuff mode or **Rollback** without knowing or storing the RCON password. The menu is Companion's built-in WASD menu. Chat-selected modes use the isolated Companion profiles and defaults from `RCT.json`. Companion captures the live server ConVar values before either interface applies a selection and owns the shared rollback snapshot.

Universal and `!fun` can roll back or replace modes applied by either interface. Every replacement first restores the shared pre-mode snapshot, then captures a fresh snapshot before applying the new mode. A server-side operation lock prevents another supported interface from applying or rolling back a mode while a change is in progress, including across Deathmatch map reloads. The lock expires after two minutes as a recovery safeguard if a client disconnects mid-operation.

## Targets

| Target | Matches |
| --- | --- |
| `@all` | All connected players and bots |
| `@ct` | Counter-Terrorists |
| `@t` | Terrorists |
| `@spec` | Spectators |
| `@alive` | Alive players |
| `@dead` | Dead players |
| `@bot` | Bots |
| `@human` | Human players |
| `#<userid>` | One server user ID, such as `#12` |
| `<SteamID64>` | One Steam account |
| `<exact name>` | One or more exact case-insensitive name matches |

Quote player names containing spaces in an RCON command.

## Examples

```text
css_rct_give @all taser
css_rct_health @ct 250
css_rct_speed "Player Name" 1.5
css_rct_gravity #12 0.5
css_rct_god @human on
css_rct_zeus_enable
css_rct_status
css_rct_zeus_disable
```

## Managed State

Speed, gravity, freeze, god mode, and noclip settings are tracked by SteamID and reapplied after spawn. `css_rct_reset` restores their normal values. Unloading the plugin also restores managed movement and damage values. Universal activates a named Companion profile for every Fun Stuff mode. A profile-disable command only succeeds when that profile still owns the active state, preventing delayed or persisted cleanup from one mode from changing another mode. ScoutzKnives, Pistols Only, Knife Arena, AWP Wars, Grenade Wars, Molotov/Incendiary Wars, and Zeus Wars use managed inventories. Headshot Only, Bhop, Deathmatch, and Surf use explicit profiles with unrestricted weapons. No Bots (`css_rct_mode_enable nobots`, first entry in `!fun`) only sets `bot_quota 0` and kicks bots; it leaves loadouts, buying, and ground cleanup untouched, re-asserts itself after map loads, and its rollback restores the captured bot quota without restarting the round.

Managed modes replace each alive human or bot inventory immediately, after future spawns, and when a human takes control of a bot, while preserving the Terrorist bomb carrier's C4. The plugin strips the old loadout through CounterStrikeSharp's supported inventory API and then grants the mode-owned items. ScoutzKnives grants one SSG 08 and the player's team knife; AWP Wars grants only an AWP. Zeus grants only a Zeus, Knife Arena grants only the team knife, and Pistols Only grants only the team's default pistol. Grenade Wars grants an HE grenade, while Molotov/Incendiary Wars grants a Molotov to Terrorists or an incendiary grenade to Counter-Terrorists. Human players retain the team knife as CS2's required post-throw fallback weapon; after a throw, the plugin restores grenade ammo and selects dedicated slot 6 for HE or slot 10 for Molotov/incendiary. Bots receive no knife in grenade modes, so their AI cannot prefer melee combat. During a bot takeover, weapon and ammo operations continue through the live bot controller while slot-selection commands are sent through the human controller. While buying is disabled, every game tick removes buy-zone access without cancelling normal weapon selection or C4 planting. Disabling managed loadouts does not strip players again; normal game loadouts return through the game mode's usual spawn or round behavior.

The plugin does not read, execute, or modify `server.cfg` or any other server configuration file.
