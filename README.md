# RCON Tool Companion v1.0.2

## Added

**In-Game Administration:**

- Added the built-in WASD `!fun` / `/fun` menu for administrators with `@css/fun`: W/S move, E selects, A returns, and R exits. The desktop RCON/Server Tool is optional for in-game Fun Stuff.
- Added the server-only `css_rct_fun_apply` command for scheduled mode rotations using `RCT.json` defaults.

**Fun Stuff Modes:**

- Added isolated **No Bots**: removes bots and holds quota at zero without changing weapons, buying, or ground cleanup. Rollback restores the captured quota without a round restart.
- Added standalone **Bot Control** directly below No Bots in `!fun`. Bot enablement, quota, quota mode, and team are staged until **Apply now**, which saves the selection to `RCT.json` and applies it without replacing the active gameplay mode.
	- **Bot team** can be Any, CT only (humans play T), or T only (humans play CT). Original team settings and active-side restore metadata survive plugin reloads.
- Added **Super Hero** with fourteen configurable powers: Speedster, Moon Jumper, Tank, Ghost, Vampire, Regenerator, Juggernaut, Second Wind, Ammo Recycler, Berserker, Medic, Blink, Double Jump, and Gunslinger. Normal weapons and buying remain available.
- Moon Jumper combines the configurable `MoonJumperJumpVelocity` takeoff value with `LowGravityScale` for slower descent.
- Added team-balanced Super Hero assignment: choose from the least-used enabled powers among living teammates before repeating one. CT and T are assigned independently, bots are tracked individually, and respawns prefer a different power when possible.
- Added configurable Super Hero power popups for players and spectators, including spectated bots. The popup shows the same name and description as chat, retains activation instructions such as Blink's Use key, and does not freeze movement. `SuperHero.PowerNoticeSeconds` defaults to five seconds; `0` disables it and the maximum is `30`.
- Added isolated **One in the Chamber**: stock Deagle plus knife, exactly one starting bullet, one bullet earned per non-self kill, one-hit kills, Companion-controlled respawning, and a per-life ammo ledger. Optional Deagle-headshots-only rules keep knife hits lethal. Respawn delay defaults to one second; standard bot controls are supported. Buying, C4 objectives, and other weapons are disabled. Chamber owns an independent native Deathmatch FFA ruleset with individual scoring and random spawns. Entry and rollback reload the current map; rollback restores captured cvars after map configs load before starting a pending replacement. Map reapply adds no round restart. Existing Deathmatch commands and timing are unchanged. Requires `chamber-native-ffa`; roll back an active older Chamber session before upgrading. Native HUD/scoring and gameplay still require live testing.

**Mode Lifecycle & Configuration:**

- Added configurable automatic rollback when the server remains empty. `RollbackWhenServerEmpty` defaults to enabled; `EmptyServerRollbackDelaySeconds` defaults to `300` (five minutes), with a range of `0`-`3600`.
- Returning humans cancel pending empty-server rollback. Repeated checks and map changes do not restart the grace deadline. Explicit Rollback remains immediate; mode replacement and unload invalidate pending callbacks.
- Added map-transition guarding and diagnostic status fields for transition state, last mode-end reason, and remaining grace time.
- Added guarded map-start reapplication for active chat/scheduled modes and Super Hero. Saved commands replay after map configs load without recapturing rollback state; Super Hero resets map-specific power, life, and cooldown state. Stale callbacks are guarded against rollback, replacement, ownership changes, and newer activations.
- Added `css_rct_empty_server_config` to persist server-wide automatic rollback and grace settings from Universal. Policy readback is exposed through `css_rct_status`; saving the policy does not start or replace a mode. Pending-delay edits preserve elapsed grace time.
- Added `BotControl`, `SuperHero`, and `OneInTheChamber` configuration sections and the top-level empty-server policy options. Missing new fields in existing `RCT.json` files use defaults; regeneration is not required.

## Changed

**Menu & Integration:**

- Kept No Bots and Bot Control first, alphabetized gameplay modes after them, and placed Rollback last.
- Removed MenuManagerAPI, MenuManager, and PlayerSettingsCS2 dependencies. The menu is implemented inside Companion.
- Kept administrative console commands server/RCON-only and the in-game menu permission-protected. The README now explains standalone use and links to the optional desktop management tool.
- Added capability tags for new modes, settings, popups, and lifecycle behavior so Universal can reject unsupported older test builds.

**Runtime & Packaging:**

- Targets .NET 10 and CounterStrikeSharp API 1.0.376. Expanded powers use native pre-damage listeners and tracing, so the full compatible server runtime is required, not only a replacement API DLL.
- RCT archives now start at `addons/counterstrikesharp/plugins/RCT/RCT.dll`. Extract into the server's `game/csgo/` directory. The deployed plugin directory and DLL remain named `RCT` and `RCT.dll`.

## Fixed

- Corrected every Companion settings save to the deployed `configs/plugins/RCT/RCT.json` path. The prior framework helper derived `RconToolCompanion/RconToolCompanion.json` from the assembly name, causing the reported hosted-server save failure. Settings Apply and JSON persistence have now been confirmed working in the user's hosted-server test.
- Corrected same-frame mode-switch rollback capture to use the restored values rather than stale outgoing ConVars while commands are queued, preventing bot quota zero from becoming the replacement mode's baseline.
- Fixed WeaponRestrict preventing ScoutzKnives and AWP modes started through `!fun` or scheduled rotation from giving players their forced weapons. Restriction commands are skipped when that plugin is absent.

**Validation & Remaining Checks:**

- Release builds and focused checks passed for power allocation, damage/jump predicates, ammo accounting, map/rollback guards, grace deadlines, popup clocks and target identity, policy conversion/readback, and RCT-path file save/reload behavior.
- User live tests confirmed hosted settings saving, Super Hero map-rotation carryover, and readable spectator power notices. These do not establish that every power, mode transition, or platform has been fully validated.
- Remaining checks include broader One in the Chamber and Super Hero gameplay, actual five-minute-empty expiry/reconnect scenarios, exact popup duration, and the residual slight HUD dimming. No claim of flicker-free rendering is made.
