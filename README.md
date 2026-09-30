# MMOSkillClassesPack

Three free baseline classes for the [MMO Skill Tree](https://www.curseforge.com/hytale/mods/mmo-skill-tree) mod (1.6.0+).

## Classes

- **Adventurer** - the classless default. No restrictions, no bonuses. Pick this if you want the pre-class experience.
- **Warrior** - frontline melee. Bonuses on melee weapons + defense; magic and artillery locked down.
- **Hunter** - bow + dagger + nature. Critical-chance and archery damage focus; staves and most magic locked.

Each class has three advancement ranks (Initiate / Adept / Master) gated by total level and key skill thresholds. Reaching an advancement grants additional passive rewards.

## Install

1. Install the [MMO Skill Tree](https://www.curseforge.com/hytale/mods/mmo-skill-tree) mod 1.6.0+.
2. Drop `MMOSkillClassesPack.zip` into your Hytale mods folder.
3. Start the server. The class system activates on first load.
4. Players use `/mmoclass list` and `/mmoclass select --id=<classId>` to pick a class.

Without this pack the class system stays dormant; the mod behaves exactly as it does with no class system at all.

## Authoring your own classes

A class is one file at `Server/MMOSkillTree/Classes/<Id>.json`. The fields sit at the top level in PascalCase, with no wrapper object, and the mod's class codec is the schema. The class id is the file name in lowercase (`Warrior.json` is `warrior`).

Display text is never written in the JSON. Add lang keys to `Server/Languages/<bcp47>/mmoskilltree.lang` instead: `class.<id>.name`, `class.<id>.flavor` and `class.<id>.desc` for the class, and `class.<id>.<rankId>.name` and `.flavor` for each advancement rank.

### Layering

- A class file with the same id as one in another pack replaces it, and `"Enabled": false` on an id switches that class off.
- A server owner has the last word in `mods/mmoskilltree/classes.json`. It holds fragments of the same shape keyed by lowercase class id under `classes`, for example `{"classes": {"warrior": {"Color": "#804040"}}}`, merged field by field over the pack's file. An id no pack ships is read as a brand new class the owner wrote. The order is: built-in (empty), then packs, then the owner file.
- To share a base between classes, mark a file `"Abstract": true` and name it from other classes with `"Parent": "<id>"`. Children inherit field by field, and `Advancements` merges per rank id. An abstract class never shows up as selectable. The three classes here stand alone, since nothing they share is worth a base; write one the day two of your own classes repeat themselves.

### Class fields

All fields are optional.

| Field | Type | Notes |
|-------|------|-------|
| `Abstract` | bool | A shared base other classes name as `Parent`. It is never selectable, and this field is never inherited. |
| `Enabled` | bool | Unwritten means true; `false` parks the class. |
| `Icon` | string | A Hytale item id used as the class icon. |
| `Color` | string | Hex accent for the class screens, such as `#a04a4a`. |
| `Requires` | requirement block | Gates selecting the class. It is the same block quests and mastery use: `Factors` are numeric bounds (a skill level is `{"Factor": "hytale:stat", "Param": "MMO_Level_<SKILL>", "Min": n}`, the total is `MMO_TotalLevel`, a mastery node is `{"Factor": "mmoskilltree:mastery_node", "Param": "<track>:<node>", "Min": 1}`), plus `Permission`, `Quests`, and `AllOf` / `AnyOf` / `Not`. Unwritten means anyone can pick it. |
| `SwitchPolicy` | group | Overrides the server's switch policy for this class (see below). |
| `Grants` | group | Applied while this class is selected (see below). |
| `Advancements` | map | Rank id to advancement, in unlock order. A rank id is what a player's unlocks are saved under, so renaming one orphans everybody who reached it. |

### Grants

The class and each advancement use the same group. An advancement's grants stack on top of the class grants and every earlier rank.

| Field | Type | Notes |
|-------|------|-------|
| `XpMultipliers` | map of skill to number | `0.0` means no XP at all, `1.25` is +25%, `1.0` is unchanged. Skill ids are uppercase. |
| `Abilities` | `{Allow, Deny}` | Arrays of ability ids in lowercase. An empty or missing `Allow` means no gate. `Deny` always wins. |
| `Mastery` | `{Allow, Deny}` | Arrays of `trackId` or `"trackId:nodeId"`. Gates mastery purchases the same way. |
| `SkillRewards` | `{Allow, Deny}` | Arrays of reward ids. Gates skill tree reward claims the same way. |
| `PassiveRewards` | array | Entries in the skill tree reward shape: `{"id", "type", "value"}`, plus optional `combatTarget`, `customCombatTargetId` and HP-range fields. Combat types (`FLAT_DAMAGE`, `FLAT_LIFESTEAL`, `FLAT_COMBO_DAMAGE`) feed the per-hit totals. `STAT_HEALTH`, `STAT_STAMINA` and `STAT_MANA` apply as max-stat modifiers. |
| `StartingItems` | map of item id to count | Handed over each time the class is selected, including a switch back to it. Only the class-level block is delivered; an advancement's `StartingItems` is not. |
| `StartingMasteryNodes` | array of `"trackId:nodeId"` | Read and merged, but nothing grants the nodes yet, so writing it has no effect in play. |
| `ClassQuests` | array of quest ids | Read and merged, but nothing reads it yet, so writing it has no effect in play. |

### Advancements

Each entry under `Advancements` takes `Icon`, `Requires` (when the rank unlocks) and `Grants` (added on top of the class grants and earlier ranks). Here is the shape, taken from the Warrior:

```json
"Advancements": {
  "initiate": { "Requires": { "Factors": [ { "Factor": "hytale:stat", "Param": "MMO_TotalLevel", "Min": 50 } ] },
                "Grants": { "PassiveRewards": [ { "id": "warrior_initiate_dmg", "type": "FLAT_DAMAGE", "value": 2 } ] } } }
```

### Switch policy

| Field | Type | Default |
|-------|------|---------|
| `FirstSwitchFree` | bool | true |
| `Cost` | group | `{"Currency": "<id>", "Amount": n, "Items": {"<itemId>": count}}` |
| `CooldownMs` | long | 24 hours |
| `EscalationMultiplier` | number | 1.0 |
| `PermanentLock` | bool | false |

### Try it

Put the mod and the built zip in your Hytale mods folder and start the server. `/mmoclass list` should show your classes, `/mmoclass select --id=warrior` swaps you in and applies the starting items and passives, and `/mmoclass advancements` shows the rank ladder and your progress. Look in the server log for `Failed to decode asset:` lines, which mean a file did not match the schema.

## Build (from source)

```powershell
.\build.ps1                  # build the zip, and install it if a Mods folder is known
.\build.ps1 -Install:$false  # build only, no copy
```

The script is self-contained and cross-platform (`pwsh ./build.ps1` works on macOS/Linux). It zips with the forward-slash plus directory entries Hytale needs; never use `Compress-Archive`. To auto-install on build, set `HYTALE_MODS_DIR` once to your Hytale `UserData/Mods` folder (or pass `-ModsDir <path>`); without it the script just builds the zip.
