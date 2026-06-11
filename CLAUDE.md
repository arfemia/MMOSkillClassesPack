# CLAUDE.md — MMOSkillClassesPack

This directory is **a standalone Hytale content pack** that ships as its own mod on CurseForge alongside the [MMOSkillTree mod](https://www.curseforge.com/hytale/mmoskilltree). The plugin's `ClassesConfig` ships no built-in classes; without this pack (or another class pack), the class system is fully dormant — every UI/command/hook routes to identity behavior, preserving byte-for-byte 1.1.7 behavior.

## Layout

```
skill-classes-pack/
├── manifest.json                                Hytale plugin manifest
├── CLAUDE.md                                    this file
├── README.md                                    end-user installation notes
├── MMOSkillClassesPack.zip                      built artifact
└── Server/
    └── MMOSkillTree/
        ├── Control/MMOSkillClassesPack.json     add/replace per content type (Classes, ClassTemplates)
        ├── ClassTemplates/*.json                1 template (ClassStandard)
        └── Classes/*.json                       3 classes (Adventurer, Warrior, Hunter)
```

## Build & deploy

```powershell
.\build.ps1                  # build the zip, and install it if a Mods folder is known
.\build.ps1 -Install:$false  # build only, no copy
.\build.ps1 -ModsDir <path>  # build + install into an explicit folder
```

`build.ps1` is self-locating and cross-platform (Windows PowerShell, or `pwsh ./build.ps1` on macOS/Linux). It zips with the `[IO.Compression.ZipFile]::Open` API using forward-slash relative paths plus an explicit directory entry per ancestor path; Hytale's asset loader silently drops the backslash-separated entries `Compress-Archive` writes, so never use it. To auto-install on build, set `HYTALE_MODS_DIR` once to your Hytale `UserData/Mods` folder (or pass `-ModsDir`); without it the script just builds the zip.

## Pack JSON conventions

**Asset key comes from the filename** (PascalCase per Hytale requirement); `Payload` is a nested JSON object captured via `Codec.BSON_DOCUMENT`. The plugin parses the inner JSON via `ClassesConfig.parseClass`. Class id = filename lowercased (e.g. `Warrior.json` → `warrior`).

## Class template extension (plugin 1.2.0+)

The `ClassStandard` template ships the rank ladder skeleton (3 advancements: Initiate / Adept / Master) with empty `baseGrants` and empty advancement grants. Concrete classes:

1. **`extends: "classstandard"`** — pull the template.
2. **`params: { ... }`** — feed `{{paramName}}` substitution (displayName, flavor, description, icon, per-advancement display name / flavor). Empty resolve drops the holding key.
3. **`baseGrantsOverrides: { ... }`** — deep-merge into the resolved `baseGrants` object. This is where the class fills its xp multipliers, ability blacklist/whitelist, passive rewards, and starting items.
4. **`advancementOverrides: { "initiate": { requirements: ..., grants: ... }, ... }`** — per-advancement deep-merge into the matching entry in `advancements[]` (matched by `id`). Per-class differentiates here: when do you reach this rank, what does it grant.
5. **`extraAdvancements: [ ... ]`** — append wholly new advancements after the template's three. Use this if a pack wants to extend the ladder beyond Master (e.g. paid Vanguard pack adds a 4th rank `legend`).

The shared substitution + deep-merge primitives are in `JsonTemplateUtil` (zero new logic per content type — same DSL Mastery/Quest/Achievement/CommandRewards uses).

## ClassDefinition schema reference

Inner Payload fields (all optional unless noted):

| Field | Type | Notes |
|-------|------|-------|
| `displayName` | string | falls back to id |
| `flavor` | string | short tagline |
| `description` | string | longer copy |
| `icon` | string | Hytale item id used as the class icon |
| `unlockRequirements` | PrerequisiteGroup | gate selection — quests, skill levels, total level, achievements, currency, etc. Empty = freely selectable. |
| `switchPolicy` | SwitchPolicy | per-class override of the global switch policy (cost, cooldown, escalation, permanent lock). null = use ClassesConfig.getGlobalSwitchPolicy(). |
| `baseGrants` | ClassGrants | applied while this class is selected (see below) |
| `advancements` | array of `ClassAdvancement` | additive ranks unlocked by meeting prereqs |

**ClassGrants** — base + each advancement use the same shape; advancements deep-merge into base for the player's effective grants:

| Field | Type | Notes |
|-------|------|-------|
| `xpMultipliers` | map<skill, double> | `0.0` = no XP gain; `1.25` = +25%; `1.0` = unchanged. Skill ids uppercased. |
| `abilityWhitelist` / `abilityBlacklist` | array<abilityId> | empty whitelist = no gate. Blacklist always wins. |
| `masteryWhitelist` / `masteryBlacklist` | array<trackId or "trackId:nodeId"> | gates mastery purchases. |
| `skillRewardWhitelist` / `skillRewardBlacklist` | array<rewardId> | gates skill-tree reward claims. |
| `passiveRewards` | array<SkillReward> | applied while the class is active; combat-typed entries plug into the per-hit FLAT_DAMAGE / FLAT_LIFESTEAL / FLAT_COMBO_DAMAGE aggregators; STAT_HEALTH / STAT_STAMINA / STAT_MANA apply as max-stat modifiers via `StatModifierUtil`. |
| `startingItems` | map<itemId, count> | given once on class selection (not on every switch). |
| `startingMasteryNodes` | array<"trackId:nodeId"> | auto-granted on selection. |
| `classQuests` | array<questId> | unlocked while this class is active. |

**ClassAdvancement**:

| Field | Type | Notes |
|-------|------|-------|
| `id` | string (required) | stable identifier — used as both the deep-merge key and the persisted unlock record. |
| `displayName` / `flavor` / `icon` | string | UI surface. |
| `requirements` | PrerequisiteGroup | when this rank unlocks. Same shape used by quests + mastery. |
| `grants` | ClassGrants | additive over baseGrants + prior advancements. |

**SwitchPolicy**:

| Field | Type | Default |
|-------|------|---------|
| `firstSwitchFree` | bool | true |
| `costCurrencyId` | string | `mastery_points` |
| `costAmount` | long | 100 |
| `costItems` | map<itemId, count> | empty |
| `cooldownMs` | long | 24h |
| `escalationMultiplier` | double | 1.0 |
| `permanentLock` | bool | false |

## Sync with plugin

The pack and the plugin co-evolve:

- If the plugin adds a new `RewardType` referenced in `passiveRewards`, re-emit any affected class JSON here.
- If the plugin extends the schema, update `ClassesConfig.SCHEMA_VERSION` and document the migration in the plugin CHANGELOG.

## Verification

1. Build the plugin: `./gradlew build` from the monorepo root, two levels up (`../../`).
2. Build the pack zip (see Build & deploy above).
3. Copy both into your Hytale mods folder.
4. Start the server. Confirm in the server log:
   - `[AssetPacks] Class pack layer applied (3 entries, mode=add) — 3 classes effective`
   - No `Failed to decode asset:` or `ClassesConfig: failed to parse pack class` lines.
5. In-game: `/mmoclass list` shows three classes; `/mmoclass select --id=warrior` swaps you in and applies the starting kit + passives. `/mmoclass advancements` reflects rank thresholds.
