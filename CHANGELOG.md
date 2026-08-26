# Changelog - MMO Skill Classes Pack

## [2.0.0] - UNRELEASED (HELD)

- **The three classes (Adventurer, Warrior, Hunter) rewritten as structured class files** (`Server/MMOSkillTree/Classes/<Id>.json`) read by the plugin's own class codec: `Icon`/`Color`/`Requires`/`SwitchPolicy`/`Grants`/`Advancements`, with rank-by-rank reuse via native `Parent`. The `ClassStandard` template file and the pack Control file are retired.
- Class content is unchanged in play: the same grants, locks, advancement ranks and switch rules.
- Requires MMO Skill Tree 1.6.0+.
