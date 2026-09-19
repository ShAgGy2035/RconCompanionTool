# RCON Tool Companion

Standalone CounterStrikeSharp companion plugin for ShAgGy's Ultimate CS2 RCON/Server Tool.

Version: 1.0.1  
Author: ShAgGy

RCON Tool Companion provides player-level fun commands that CS2's native RCON console cannot perform reliably. It is independent of SimpleAdmin and every command is restricted to the server console or RCON.

## Requirements

- A Counter-Strike 2 dedicated server
- Metamod:Source
- CounterStrikeSharp 1.0.374 or the `v1.0.374-khs-khook` KHS build
- MenuManagerAPI 1.0.3 for WASD navigation in `!fun` (optional; the built-in chat menu remains available as a fallback)

## Installation

1. Extract the release ZIP into the CS2 server root.
2. Confirm this file exists:
	`game/csgo/addons/counterstrikesharp/plugins/RCT/RCT.dll`
3. Restart the server or load the plugin through CounterStrikeSharp.
4. Run `css_rct_status` through RCON. A successful response starts with `[RconToolCompanion] Ready.`

This release targets .NET 10 and CounterStrikeSharp API 1.0.374. If `css_plugins list` shows `Unknown (Unknown)`, remove the older RCT DLL, install this complete release, and restart the server instead of hot-reloading across the framework change.

Install the complete MenuManagerAPI 1.0.3 release into the server root before loading RCT. RCT references its `MenuManagerAPI.Shared.dll` contract during plugin registration and uses the `menu:api` capability for WASD navigation. If it is missing, CounterStrikeSharp reports RCT as `UNREGISTERED` or `Unknown`, and `css_rct_status` remains unavailable. MenuManager 1.5.0 and newer use a different capability and are not a drop-in replacement for this integration.

The short `RCT` plugin directory is intentional. On affected Linux container filesystems, the .NET host cannot canonicalize a longer plugin path even though CounterStrikeSharp can discover it. The DLL retains the full product name for clarity.

On first load, CounterStrikeSharp generates the standalone mode configuration at:

`game/csgo/addons/counterstrikesharp/configs/plugins/RCT/RCT.json`

Its mode sections mirror Universal's Fun Stuff options. ScoutzKnives exposes air acceleration, gravity, bunnyhopping, automatic bunnyhopping, fall-damage scale, and friction. Surf exposes cheats, acceleration, air acceleration, gravity, both bunnyhop switches, jump and landing stamina costs, round time, and freeze time. Grenade Wars, Fire Grenade Wars, and Zeus Wars always enforce their required managed loadouts and no longer expose a `ManagedLoadout` setting. Every mode also has `BotsEnabled`, `BotQuota`, and `BotQuotaMode`. `BotsEnabled` defaults to `true`; `BotQuota` defaults to `null`, which preserves the server's current quota. Set `BotQuota` from 1 to 64 to enforce an exact count for that mode. `BotQuotaMode` accepts `normal`, `fill`, or `match` and defaults to `normal`; `fill` replaces bots as human players join to maintain the configured total quota. When bots are disabled, RCT ignores the quota and quota mode, sets `bot_quota 0`, and kicks current bots. Rollback or a mode switch restores every changed cvar, including bot and physics values, to its captured pre-mode value. Edit the JSON while the plugin is unloaded, then load or restart RCT. Invalid values are clamped to supported ranges.

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

Grant trusted administrators the custom CounterStrikeSharp permission `@css/fun`. Those administrators can type `!fun` or `/fun` in chat and select any Fun Stuff mode or **Rollback** without knowing or storing the RCON password. With MenuManagerAPI 1.0.3 installed, the menu uses its WASD button navigation; otherwise, Companion reports the missing integration and opens CounterStrikeSharp's numbered chat menu. Chat-selected modes use the isolated Companion profiles and defaults from `RCT.json`. Companion captures the live server ConVar values before either interface applies a selection and owns the shared rollback snapshot.

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

Speed, gravity, freeze, god mode, and noclip settings are tracked by SteamID and reapplied after spawn. `css_rct_reset` restores their normal values. Unloading the plugin also restores managed movement and damage values. Universal activates a named Companion profile for every Fun Stuff mode. A profile-disable command only succeeds when that profile still owns the active state, preventing delayed or persisted cleanup from one mode from changing another mode. ScoutzKnives, Pistols Only, Knife Arena, AWP Wars, Grenade Wars, Molotov/Incendiary Wars, and Zeus Wars use managed inventories. Headshot Only, Bhop, Deathmatch, and Surf use explicit profiles with unrestricted weapons.

Managed modes replace each alive human or bot inventory immediately, after future spawns, and when a human takes control of a bot, while preserving the Terrorist bomb carrier's C4. The plugin strips the old loadout through CounterStrikeSharp's supported inventory API and then grants the mode-owned items. ScoutzKnives grants one SSG 08 and the player's team knife; AWP Wars grants only an AWP. Zeus grants only a Zeus, Knife Arena grants only the team knife, and Pistols Only grants only the team's default pistol. Grenade Wars grants an HE grenade, while Molotov/Incendiary Wars grants a Molotov to Terrorists or an incendiary grenade to Counter-Terrorists. Human players retain the team knife as CS2's required post-throw fallback weapon; after a throw, the plugin restores grenade ammo and selects dedicated slot 6 for HE or slot 10 for Molotov/incendiary. Bots receive no knife in grenade modes, so their AI cannot prefer melee combat. During a bot takeover, weapon and ammo operations continue through the live bot controller while slot-selection commands are sent through the human controller. While buying is disabled, every game tick removes buy-zone access without cancelling normal weapon selection or C4 planting. Disabling managed loadouts does not strip players again; normal game loadouts return through the game mode's usual spawn or round behavior.

The plugin does not read, execute, or modify `server.cfg` or any other server configuration file.
