# RCON Tool Companion

Standalone CounterStrikeSharp companion plugin for ShAgGy's Ultimate CS2 RCON/Server Tool.

Version: 1.0.0  
Author: ShAgGy

RCON Tool Companion provides player-level fun commands that CS2's native RCON console cannot perform reliably. It is independent of SimpleAdmin and every command is restricted to the server console or RCON.

## Requirements

- A Counter-Strike 2 dedicated server
- Metamod:Source
- CounterStrikeSharp 1.0.346 or newer

## Installation

1. Extract the release ZIP into the CS2 server root.
2. Confirm this file exists:
	`game/csgo/addons/counterstrikesharp/plugins/RCT/RconToolCompanion.dll`
3. Restart the server or load the plugin through CounterStrikeSharp.
4. Run `css_rct_status` through RCON. A successful response starts with `[RconToolCompanion] Ready.`

The short `RCT` plugin directory is intentional. On affected Linux container filesystems, the .NET host cannot canonicalize a longer plugin path even though CounterStrikeSharp can discover it. The DLL retains the full product name for clarity.

No configuration file or admin plugin is required.

## Commands

| Command | Description |
| --- | --- |
| `css_rct_status` | Report version, Zeus state, and capabilities |
| `css_rct_version` | Report the plugin version |
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
| `css_rct_pistols_enable` | Enable managed team-pistol-only loadouts now and on spawn |
| `css_rct_grenades_enable` | Enable managed HE-grenade-only loadouts now and on spawn |
| `css_rct_fire_grenades_enable` | Enable managed Molotov/incendiary-only loadouts now and on spawn |
| `css_rct_loadout_disable` | Disable the active managed weapon loadout |
| `css_rct_zeus_disable` | Compatibility alias for disabling managed loadouts |
| `css_rct_buy_disable` | Close buy menus, remove buy-zone access, and block purchases |
| `css_rct_buy_enable` | Restore normal buy-zone and purchase behavior |

Weapon names may be supplied with or without the `weapon_` prefix. For example, `taser` and `weapon_taser` are equivalent.

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

Speed, gravity, freeze, god mode, and noclip settings are tracked by SteamID and reapplied after spawn. `css_rct_reset` restores their normal values. Unloading the plugin also restores managed movement and damage values.

Managed Zeus, Knife Arena, Pistols Only, Grenade Wars, and Molotov/Incendiary Wars modes replace each alive human or bot inventory immediately, after future spawns, and when a human takes control of a bot, while preserving the Terrorist bomb carrier's C4. The plugin strips the old loadout through CounterStrikeSharp's supported inventory API, waits one game frame for removal to complete, then restores C4 when needed and grants the managed item. Zeus grants only a Zeus, Knife Arena grants only the team knife, and Pistols Only grants only the team's default pistol. Grenade Wars grants an HE grenade, while Molotov/Incendiary Wars grants a Molotov to Terrorists or an incendiary grenade to Counter-Terrorists. Human players retain the team knife as CS2's required post-throw fallback weapon; after a throw, the plugin restores grenade ammo and selects dedicated slot 6 for HE or slot 10 for Molotov/incendiary. Bots receive no knife, so their AI cannot prefer melee combat. After each bot throw, the plugin uses CounterStrikeSharp's supported delayed removal and grants a new managed grenade only after removal completes. During a bot takeover, weapon and ammo operations continue through the live bot controller while slot-selection commands are sent through the human controller, converting the controlled pawn to the human loadout and replenishment path. While buying is disabled, every game tick removes buy-zone access without cancelling normal weapon selection or C4 planting. Human buy menus are closed when buy blocking begins or a player spawns, while buy, autobuy, rebuy, buy-menu, and ammo-purchase commands are blocked as a transaction backstop. Disabling managed loadouts does not strip players again; normal game loadouts return through the game mode's usual spawn or round behavior.

The plugin does not read, execute, or modify `server.cfg` or any other server configuration file.
