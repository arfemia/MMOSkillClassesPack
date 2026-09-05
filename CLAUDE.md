# CLAUDE.md - MMOSkillClassesPack

This directory is **a standalone Hytale content pack** that ships as its own mod on CurseForge alongside the [MMOSkillTree mod](https://www.curseforge.com/hytale/mods/mmo-skill-tree). The plugin's `ClassesConfig` ships no built-in classes; without this pack (or another class pack), the class system is fully dormant - every UI/command/hook routes to identity behavior. The Class System itself is unreleased "Future" work in the plugin's own changelog; this pack rides the same status.

## Layout

```
skill-classes-pack/
├── manifest.json                                Hytale plugin manifest
├── build.ps1                                    zip + optional install (see below)
├── CLAUDE.md                                    this file
├── CHANGELOG.md                                 per-version pack changelog
├── README.md                                    end-user installation notes
├── MMOSkillClassesPack.zip                      built artifact
└── Server/
    ├── MMOSkillTree/
    │   └── Classes/*.json                       3 classes (Adventurer, Warrior, Hunter), structured
    └── Languages/<bcp47>/mmoskilltree.lang      class + advancement display text, 9 locales
```

## Build & deploy

```powershell
.\build.ps1                  # build the zip, and install it if a Mods folder is known
.\build.ps1 -Install:$false  # build only, no copy
.\build.ps1 -ModsDir <path>  # build + install into an explicit folder
```

`build.ps1` is self-locating and cross-platform (Windows PowerShell, or `pwsh ./build.ps1` on macOS/Linux). It zips with the `[IO.Compression.ZipFile]::Open` API using forward-slash relative paths plus an explicit directory entry per ancestor path; Hytale's asset loader silently drops the backslash-separated entries `Compress-Archive` writes, so never use it. To auto-install on build, set `HYTALE_MODS_DIR` once to your Hytale `UserData/Mods` folder (or pass `-ModsDir`); without it the script just builds the zip.

## Pack JSON conventions

A class is one STRUCTURED file at `Server/MMOSkillTree/Classes/<Id>.json` - PascalCase fields at the top level, no `Name`/`Payload` wrapper, decoded by the plugin's `ClassDefinitionAsset` codec (the codec IS the schema). Class id = filename lowercased (`Warrior.json` is `warrior`). The store merges by id natively: a later-loaded pack's same-id file replaces one of these, and `"Enabled": false` on an id switches it off. A server owner gets the last word: `mods/mmoskilltree/classes.json` (schema v2) is an owner layer of the SAME `ClassDefinitionAsset` fragments, keyed by lowercase class id under `classes` (`{"classes": {"warrior": {"Color": "#804040"}}}`) and merged per leaf over the pack's file, so the effective precedence is `defaults (empty) < pack < owner`; an id no pack ships decodes standalone as a net-new owner-authored class. Reuse is native `Parent` inheritance: a file marked `"Abstract": true` is a shared skeleton other classes name with `"Parent": "<id>"`, inheriting per leaf (`Advancements` merges per rank id). The three shipped classes are each self-contained - nothing worth sharing differs from the defaults - so no base file ships; author one the day two of your own classes repeat themselves.

## ClassDefinition schema reference

Top-level fields (all optional; display text is localization keys by convention - `class.<id>.name|flavor|desc`, `class.<classId>.<advId>.name|flavor` - never a literal in the JSON):

| Field | Type | Notes |
|-------|------|-------|
| `Abstract` | bool | marks a shared base other classes `Parent`; never becomes selectable, never inherits |
| `Enabled` | bool | unauthored means true; `false` parks the class |
| `Icon` | string | Hytale item id used as the class icon |
| `Color` | string | hex accent for the class UI, e.g. `#a04a4a` |
| `Requires` | Requires block | gate on SELECTING the class - the same shared block quests and mastery gate on (`Factors` numeric bounds: a skill level is `{"Factor":"hytale:stat","Param":"MMO_Level_<SKILL>","Min":n}`, a total `MMO_TotalLevel`, a mastery node `{"Factor":"mmoskilltree:mastery_node","Param":"<track>:<node>","Min":1}`; plus `Permission`, `Quests`, and `AllOf`/`AnyOf`/`Not`). Unauthored = freely selectable |
| `SwitchPolicy` | group | per-class override of the global switch policy (see below); unauthored uses the server's |
| `Grants` | group | applied while this class is selected (see below) |
| `Advancements` | map | rank id to advancement, in unlock order; merges per rank id under `Parent`. A rank id is what a player's unlocks are saved under - renaming one orphans everybody who reached it |

**`Grants`** - the class and each advancement use the same group; advancement grants stack additively over the class grants and every earlier rank:

| Field | Type | Notes |
|-------|------|-------|
| `XpMultipliers` | map<skill, double> | `0.0` = no XP gain; `1.25` = +25%; `1.0` = unchanged. Skill ids uppercased. |
| `Abilities` | group `{Allow, Deny}` | both are arrays of ability ids, lowercase. Empty/absent `Allow` = no gate. `Deny` always wins. |
| `Mastery` | group `{Allow, Deny}` | both are arrays of `trackId` or `"trackId:nodeId"`; gates mastery purchases the same way. |
| `SkillRewards` | group `{Allow, Deny}` | both are arrays of reward ids; gates skill-tree reward claims the same way. |
| `PassiveRewards` | array | entries in the skill-tree reward shape (`{"id", "type", "value"}` plus optional `combatTarget` / `customCombatTargetId` / HP-range fields); combat-typed entries plug into the per-hit FLAT_DAMAGE / FLAT_LIFESTEAL / FLAT_COMBO_DAMAGE aggregators; STAT_HEALTH / STAT_STAMINA / STAT_MANA apply as max-stat modifiers. |
| `StartingItems` | map<itemId, count> | handed over on every selection of this class, a switch back to it included. Only the class-level block is delivered; an advancement's `StartingItems` is not. |
| `StartingMasteryNodes` | array<"trackId:nodeId"> | decoded and merged into the class grants, but nothing grants the nodes yet - authoring it has no effect in play. |
| `ClassQuests` | array<questId> | decoded and merged into the class grants, but no quest-availability path reads it yet - authoring it has no effect in play. |

**`Advancements` entry**:

| Field | Type | Notes |
|-------|------|-------|
| `Icon` | string | UI surface. |
| `Requires` | Requires block | when this rank unlocks. |
| `Grants` | group | additive over the class `Grants` + prior ranks. |

**`SwitchPolicy`**:

| Field | Type | Default |
|-------|------|---------|
| `FirstSwitchFree` | bool | true |
| `Cost` | group | `{"Currency": "<id>", "Amount": n, "Items": {"<itemId>": count}}` |
| `CooldownMs` | long | 24h |
| `EscalationMultiplier` | double | 1.0 |
| `PermanentLock` | bool | false |

## Sync with plugin

The pack and the plugin co-evolve:

- If the plugin adds a new `SkillRewardType` constant worth using in `PassiveRewards`, re-emit any affected class JSON here.
- If the plugin extends the class schema (`ClassDefinitionAsset`), update `ClassesConfig.SCHEMA_VERSION` there and document the migration in the plugin CHANGELOG.

## Verification

1. Build the plugin: `./gradlew build` from the monorepo root, two levels up (`../../`).
2. Build the pack zip (see Build & deploy above).
3. Copy both into your Hytale mods folder.
4. Start the server. Confirm in the server log:
   - `[AssetPacks] Class asset layer applied (3 entries) - 3 classes effective`
   - No `Failed to decode asset:` or `ClassesConfig: skipping class body` lines.
5. In-game: `/mmoclass list` shows three classes; `/mmoclass select --id=warrior` swaps you in and applies the starting kit + passives. `/mmoclass advancements` reflects rank thresholds.
