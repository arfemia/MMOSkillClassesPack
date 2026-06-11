# MMOSkillClassesPack

Three free baseline classes for the [MMO Skill Tree](https://www.curseforge.com/hytale/mmoskilltree) mod (1.2.0+).

## Classes

- **Adventurer** — the classless default. No restrictions, no bonuses. Pick this if you want the pre-class experience.
- **Warrior** — frontline melee. Bonuses on melee weapons + defense; magic and artillery locked down.
- **Hunter** — bow + dagger + nature. Critical-chance and archery damage focus; staves and most magic locked.

Each class has three advancement ranks (Initiate / Adept / Master) gated by total level and key skill thresholds. Reaching an advancement grants additional passive rewards.

## Install

1. Install the [MMO Skill Tree](https://www.curseforge.com/hytale/mmoskilltree) mod 1.2.0+.
2. Drop `MMOSkillClassesPack.zip` into your Hytale mods folder.
3. Start the server. The class system activates on first load.
4. Players use `/mmoclass list` and `/mmoclass select --id=<classId>` to pick a class.

Without this pack the class system stays dormant; the mod behaves exactly as 1.1.7.

## Build (from source)

```powershell
.\build.ps1                  # build the zip, and install it if a Mods folder is known
.\build.ps1 -Install:$false  # build only, no copy
```

The script is self-contained and cross-platform (`pwsh ./build.ps1` works on macOS/Linux). It zips with the forward-slash plus directory entries Hytale needs; never use `Compress-Archive`. To auto-install on build, set `HYTALE_MODS_DIR` once to your Hytale `UserData/Mods` folder (or pass `-ModsDir <path>`); without it the script just builds the zip.
