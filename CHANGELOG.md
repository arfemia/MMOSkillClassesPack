# Changelog - MMO Skill Classes Pack

## [2.0.0] - UNRELEASED (HELD)

- **The three classes (Adventurer, Warrior, Hunter) rewritten as structured class files** (`Server/MMOSkillTree/Classes/<Id>.json`) read by the plugin's own class codec: `Icon`/`Color`/`Requires`/`SwitchPolicy`/`Grants`/`Advancements`, with rank-by-rank reuse via native `Parent`. The `ClassStandard` template file and the pack Control file are retired.
- **Class gates are nested `Grants` groups**: `Abilities` / `Mastery` / `SkillRewards`, each carrying optional `Allow` and `Deny` ability-id lists. Hunter and Warrior deny the mage kit through `Grants.Abilities.Deny`, and every denied id names a real shipped ability (`fireball`, `meteor`, `frost_nova`, `magic_passive_pyromaniac`, `arcane_missiles`, plus Hunter's `flame_stream` and `magic_capstone_arcane_resonance`).
- Class content is otherwise unchanged in play: the same grants, locks, advancement ranks and switch rules.
- Requires MMO Skill Tree 1.6.0+.
