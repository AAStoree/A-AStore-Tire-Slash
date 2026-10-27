[README.md](https://github.com/user-attachments/files/34050852/README.md)
# A&A Store — Tire Slash

Version **1.0.0** · FiveM · `cerulean` · `lua54`

A player holding a knife punctures one tire at a time. They stand beside that tire, pass a short timing minigame, and play a slash animation. The other tires stay intact.

The resource folder must be named `aastore_tireslash`. Do not `ensure` a parent folder whose name contains `&`.

There is no SQL file. Tire state is the vehicle's live tire state.

After Tebex asset escrow, buyers can still edit `config.lua`, `locales/*.json`, and this README. The other scripts are protected.

## Requirements

| Requirement | Status |
| --- | --- |
| OneSync | Required |
| QB-Core, Qbox, ESX Legacy, or ox_core | One framework |
| ox_inventory, qb-inventory, or ESX inventory | One inventory |
| ox_target or qb-target | Optional. `auto` uses a target resource when one is started, otherwise the E key |
| oxmysql | Not used |
| ox_lib | Not required. Qbox notifications normally use it. If that export fails, this resource shows its own on-screen message |

Start the framework and the inventory before this resource.

```cfg
ensure qb-core
ensure ox_inventory
ensure aastore_tireslash
```

Qbox:

```cfg
ensure qbx_core
ensure ox_inventory
ensure aastore_tireslash
```

ESX:

```cfg
ensure es_extended
ensure ox_inventory
ensure aastore_tireslash
```

Start `ox_target` or `qb-target` above this resource when you want eye-target options. `Config.Framework` and `Config.Inventory` default to `auto`.

## How to use

1. Equip the knife, switchblade, or dagger.
2. Stand next to one tire. The default range is 2.0 meters.
3. Use the target option, or press E when the prompt is showing.
4. Press E while the marker is in the green zone.
5. The slash animation plays and that tire bursts.

Escape or Backspace cancels the minigame. A flat tire shows a message and does not start again.

The action also refuses players who are in a vehicle, dead, ragdolled, or marked down or cuffed by the framework. It refuses moving vehicles, blocked classes and models, unburstable tires, a tire someone else is already slashing, and attempts faster than `Config.RateLimit`.

## Tires

| Id | Index | Bone | Alternate | Meaning |
| --- | --- | --- | --- | --- |
| `fl` | 0 | `wheel_lf` | — | Front left |
| `fr` | 1 | `wheel_rf` | — | Front right |
| `ml` | 2 | `wheel_lm1` | `wheel_lm` | Middle left, when `EnableExtraWheels` is true |
| `mr` | 3 | `wheel_rm1` | `wheel_rm` | Middle right, when `EnableExtraWheels` is true |
| `rl` | 4 | `wheel_lr` | — | Rear left |
| `rr` | 5 | `wheel_rr` | — | Rear right |

Cars without a middle axle do not show the middle options. Class 8 motorcycles are allowed. Their tire indexes can differ by model. Classes 13, 14, 15, 16, 18, and 21 are blocked by default.

## Distance

- The prompt and the target option use `Config.WheelSelectDistance` (default 2.0 m).
- The server allows that distance plus `Config.DistanceSlack` (default 0.8 m).
- The hard cap is 5.0 m. Higher config values are clamped. The cap is not editable from `config.lua`.
- `Config.ScanRadius` (default 10 m, range 6–15) only finds vehicles. The slash is measured from the tire, so a rear tire works when the player is beside it.
- The player must also be near the vehicle origin. That allowance grows slightly with vehicle size and never exceeds 12 m.

## Frameworks

`auto` checks Qbox, then QB-Core, then ESX, then ox_core. When both Qbox and QB-Core are started, Qbox wins. A forced framework is never replaced.

Accepted values: `auto`, `qbcore`, `qbox`, `esx`, `ox`. Aliases `qb-core`, `qb_core`, `qbx`, `qbx_core`, `es_extended`, and `esx_core` are normalized.

| Framework | Resource | Integration |
| --- | --- | --- |
| QB-Core | `qb-core` | `GetCoreObject`, `QBCore:Notify`, citizenid, death / last stand / cuff metadata |
| Qbox | `qbx_core` | `GetPlayer` and `Notify`, with an on-screen fallback |
| ESX Legacy | `es_extended` | `getSharedObject`, plus the legacy shared-object event |
| ox_core | `ox_core` | `GetPlayer`, ox_lib notify when that resource is started |

## Inventories

`auto` uses ox_inventory when `ox_inventory` is started. Otherwise it uses the inventory that matches the framework. A forced inventory that is not available does not silently switch to another one.

| Inventory | Config | Knife | Switchblade | Dagger |
| --- | --- | --- | --- | --- |
| ox_inventory | `ox_inventory` | `WEAPON_KNIFE` | `WEAPON_SWITCHBLADE` | `WEAPON_DAGGER` |
| QB / qb-inventory | `qb_inventory` | `weapon_knife` | `weapon_switchblade` | `weapon_dagger` |
| ESX inventory or loadout | `esx_inventory` | `weapon_knife` | `weapon_switchblade` | `weapon_dagger` |

ESX also accepts the matching `WEAPON_*` name in the weapon loadout. Set `Config.RequireInventoryItem = false` only when you want to skip the item check.

Knife, switchblade, and dagger start enabled. Set `enabled = false` to block one. All three item name fields are required. A knife entry with a missing item name is ignored. If every knife is invalid, the resource does not become active.

`Config.ServerWeaponCheck = true` reads the ped weapon on the server. That native is sometimes still unarmed while the player is holding the knife. In that moment a configured knife in inventory is accepted. A different weapon in hand is rejected. Set the check to `false` only after you decide the inventory check is enough.

## Target and the E key

```lua
Config.Interaction = 'auto' -- 'target' or 'key'
Config.Target = 'auto'      -- 'ox_target', 'qb_target', or 'none'
```

`auto` prefers ox_target, then qb-target, then the E key. `target` keeps the E key working until the selected target resource starts, and does not load the other target. `key` never registers a target, including when a target resource starts later. `Config.Target = 'none'` skips target lookup.

The E prompt is shown only on foot, with a knife equipped, beside a tire in range. Target mode hides that prompt. Options are bound to wheel bones, so a missing bone means no option for that tire.

## Languages

```lua
Config.Locale = 'en' -- or 'tr'
```

Strings live in `locales/en.json` and `locales/tr.json`. A missing Turkish key falls back to English. Restart the resource after editing them. Keep `%s` and `%d` placeholders.

Notifications use the framework notify first. If that call fails, this resource shows its own toast.

## Blocked vehicles

| Class | Vehicles |
| --- | --- |
| 13 | Cycles |
| 14 | Boats |
| 15 | Helicopters |
| 16 | Planes |
| 18 | Emergency, including police, ambulance, fire, and emergency-class addons |
| 21 | Trains |

Remove `18` only when ambulance and fire vehicles should become slashable. Keep the police spawn names in `Config.BlockedModels`.

Default models: `police`, `police2`, `police3`, `police4`, `police5`, `policeb`, `policeold1`, `policeold2`, `policet`, `polgauntlet`, `sheriff`, `sheriff2`, `fbi`, `fbi2`, `pranger`, `riot`, `riot2`.

Add addon police **spawn names**, not display names. `Config.RespectUnburstable = true` refuses tires another script marked unburstable. Vehicles faster than `Config.MaxVehicleSpeed` (default 2.0 m/s, range 0–15) cannot be slashed.

## Minigame

The server places the zone and checks the press. An instant success packet is rejected. A modified client can still time the zone it is allowed to see. Distance, weapon, item, vehicle, and duplicate-action checks stay on the server.

| Option | Default | Range | Meaning |
| --- | --- | --- | --- |
| `Period` | 1600 | 900–4000 | Milliseconds for the marker to travel one way |
| `ZoneWidth` | 0.16 | 0.08–0.28 | Width of the green zone |
| `Grace` | 70 | 0–120 | Hidden latency forgiveness, milliseconds |
| `MaxClockSkew` | 250 | 50–500 | Accepted difference between the player clock and the server clock |
| `MinReaction` | 80 | 0–400 | Presses earlier than this are rejected |
| `Timeout` | 12000 | 3000–20000 | Time allowed to finish the round |
| `Rounds` | 1 | 1–3 | Rounds that must be passed in a row |

Out-of-range numbers are clamped and a console line is printed.

`Config.Control` defaults to `38`. `Config.KeyLabel` defaults to `E` and is only the on-screen label. Change both together. The label is capped at 12 characters.

## Animation

Primary dict `anim@amb@clubhouse@tutorial@bkr_tut_ig3@`, clip `machinic_loop_mechandplayer`. If it does not load, the medic fallback is used and debug logs that failure.

| Option | Default | Range |
| --- | --- | --- |
| `Duration` | 1800 | 800–8000 |
| `TurnTime` | 350 | 0–2000 |
| `Flag` | 1 | 0–51 |

`Config.FreezeDuringSlash` keeps the player at the tire until the animation ends. Freeze is cleared on resource stop. Finish packets that arrive before about 75 percent of the animation time are rejected.

`Config.RequireLineOfSight` defaults to `false`. Turn it on only after you try it on your map. Body panels and doorways can cause false refusals.

## Discord log

Do not put a webhook in `config.lua`. That file is shared with clients. Use a server-only convar:

```cfg
set aastore_tireslash_webhook "https://discord.com/api/webhooks/1234567890/token"
```

Use `set`, not `setr`. Leave it unset to disable the log. The URL must be a Discord webhook. A bad URL is reported once and is not called. `Config.LogSlashes = true` prints one server-console line and does not send data anywhere.

## Developers

Listen on the server with `AddEventHandler`. Do not register this as a net event.

```lua
AddEventHandler('aastore_tireslash:server:tireSlashed', function(payload)
    -- payload.source, payload.netId, payload.tireIndex, payload.tireId
    -- payload.plate, payload.identifier, payload.coords = { x, y, z }
end)
```

`tireId` is `fl`, `fr`, `rl`, `rr`, and `ml` or `mr` when extra wheels are enabled.

```lua
exports.aastore_tireslash:IsSlashing()
exports.aastore_tireslash:HasKnifeEquipped()
```

`AAStoreIsSlashing` and `AAStoreHasKnifeEquipped` are the same exports. `IsSlashing` is true while a request, the minigame, or the animation is in progress.

## Configuration reference

Invalid numbers are clamped. An invalid framework, inventory, or interaction becomes `auto`. With no valid knife, the resource stays inactive.

| Option | Default | Purpose |
| --- | --- | --- |
| `Locale` | `en` | `en` or `tr` |
| `Framework` | `auto` | `auto`, `qbcore`, `qbox`, `esx`, `ox` |
| `Inventory` | `auto` | `auto`, `ox_inventory`, `qb_inventory`, `esx_inventory` |
| `Interaction` | `auto` | `auto`, `target`, `key` |
| `Target` | `auto` | `auto`, `ox_target`, `qb_target`, `none` |
| `Debug` | `false` | Extra logs, a tire marker in key mode, and `/aastore_tireslash_debug` |
| `LogSlashes` | `false` | One console line after a successful slash |
| `Control` / `KeyLabel` | `38` / `E` | Game control and the label shown on screen |
| `WheelSelectDistance` | `2.0` | Tire selection range in meters, 0.6–5.0 |
| `DistanceSlack` | `0.8` | Extra server meters, 0–1.5 |
| `ScanRadius` | `10.0` | Vehicle search radius, 6–15 |
| `MaxVehicleSpeed` | `2.0` | Speed ceiling in m/s, 0–15 |
| `RateLimit` | `1500` | Minimum delay between attempts, 500–15000 ms |
| `ServerWeaponCheck` | `true` | Server also looks at the held weapon |
| `RequireInventoryItem` | `true` | Player must own the matching item |
| `FreezeDuringSlash` | `true` | Freeze the player during the animation |
| `EnableExtraWheels` | `true` | Middle tires on vehicles that have those bones |
| `RespectUnburstable` | `true` | Refuse tires marked unburstable |
| `RequireLineOfSight` | `false` | Optional wall check |
| `BlockedClasses` | 13, 14, 15, 16, 18, 21 | Blocked vehicle classes |
| `BlockedModels` | Vanilla police names | Blocked spawn names |
| `Knives` | Knife, switchblade, dagger | Allowed blades |
| `Minigame` | See the table above | Timing minigame |
| `Animation` | See the table above | Slash animation |

`/aastore_tireslash_debug` exists only while `Config.Debug` is true. It prints the weapon hash, whether it is a configured knife, the nearest tire id, and the distance.

## Files

```text
aastore_tireslash/
  fxmanifest.lua
  config.lua
  README.md
  bridge/framework/
  bridge/inventory/
  bridge/target/
  client/main.lua
  server/main.lua
  shared/constants.lua
  shared/locale.lua
  shared/utils.lua
  locales/en.json
  locales/tr.json
  web/index.html
  web/css/style.css
  web/js/app.js
```

The NUI uses Segoe UI, then Arial. It does not request a web font.

## Performance

Without a knife, loops sleep about 250–400 ms. In key mode, a knife scans nearby tires about four times a second, and the prompt loop runs every frame so E can be read. Target mode does not run that scan. The minigame and the animation run every frame only while they are active.

These costs were not measured with resmon. Expect the idle resource to sit low on the client time list, and expect a small cost while the prompt, minigame, or animation is active.

## Troubleshooting

| Problem | What to check |
| --- | --- |
| Resource does not become ready | Framework and inventory must start first. The server console names the missing dependency |
| Prompt or target option never appears | Knife in hand, on foot, within 2.0 m, vehicle not blocked and not moving. In `auto`, a started target resource hides the E prompt |
| Minigame opens but the tire does not burst | OneSync must be on. The player has to stay through the animation. Early finishes are logged when debug is on |
| Honest players are told they are not holding a knife | The server weapon native can lag. Set `ServerWeaponCheck` to `false` only if the inventory check is enough |
| "You do not have a knife" | Item name mismatch. ox uses `WEAPON_KNIFE`. QB uses `weapon_knife` |
| A police addon can still be slashed | Add its spawn name to `BlockedModels`, or set its class to 18 |
| Target option missing | `Config.Target` may be forcing the wrong resource. That wheel bone has to exist |
| Turkish text is missing | Restart after editing `locales/tr.json`. Missing keys fall back to English |
| Discord log never arrives | Use `set`, not `setr`. The URL must be a Discord webhook. Do not put it in `config.lua` |
| Two players slash the same tire | The second player is told that tire is busy. The lock clears on cancel, failure, timeout, drop, and resource stop |

Set `Config.Debug = true` and restart. Read the server console and the client F8 console together.

## Security

Clients are not trusted for success, distance, item counts, or which tire bursts. The tire index is checked against the configured wheel list. The server checks the ped, class, model, speed, death or cuff state, distance, weapon, and inventory. One player has one session. One tire is locked by one player. Early finish packets are rejected. Sessions and target options are cleared on stop. No telemetry is sent. The Discord webhook is optional and read from a server convar.

A game client that owns the vehicle entity can still call the game's own tire native. That limit belongs to GTA networking.

## Known limitations

- The burst is applied by the slashing player and the entity owner after the script requests network control. Other players see it through OneSync.
- If the server cannot read a wheel bone, distance falls back to 5.0 m from the vehicle and that warning is printed once.
- Repair scripts that fix tires locally will unburst them for players who can already see the vehicle. This resource does not store a permanent burst record.
- Motorcycle bones vary by model.
- Line of sight is off by default.
- Qbox notifications normally depend on ox_lib.
- While the server weapon native still reports unarmed, a configured knife in inventory is accepted. A different weapon in hand is rejected.

## Changelog

### 1.0.0

- Initial A&A Store release.
- Per-tire knife slashing, timing minigame, and slash animation.
- QB-Core, Qbox, ESX, and ox_core. Framework, inventory, and interaction default to `auto`.
- ox_inventory, QB inventory, and ESX inventory adapters.
- ox_target and qb-target, with the E key when neither target resource is started.
- An unsynced unarmed ped does not block a player who is holding a configured knife. A different weapon in hand is still rejected.
- English and Turkish text.
