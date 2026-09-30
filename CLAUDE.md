# Skill Classes Pack

Class definitions for the Class System (unreleased). The family-wide rules apply here; this file adds only what is specific to this pack.

- The jar ships no built-in classes, so without a class pack the whole class system is dormant.
- An advancement's rank id is what players' unlocks are saved under; renaming one orphans everyone who reached it.
- `Grants.StartingMasteryNodes` and `Grants.ClassQuests` decode and merge, but nothing consumes them yet: authoring them has no in-game effect.
- Only the class-level `StartingItems` is handed over (on every selection, a switch back included); an advancement's `StartingItems` is never delivered.
- `Grants.PassiveRewards` entries keep the lowercase skill-tree reward shape (`id`, `type`, `value`), unlike the PascalCase fields around them.
