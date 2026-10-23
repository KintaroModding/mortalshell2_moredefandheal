# More Defense and Heal

Each heal charge restores 70 instead of 40 health, and the player takes about half the damage (base damage reduction
0.5 instead of 0).

A mod for Mortal Shell II (Unreal Engine 5.6), made with game version `1.12.0`. This README is
for developers. What the mod does, how to install it and its compatibility are in [openmods.md](openmods.md), the mod
page on [openmods](https://openmods.nexus).

## How it works

The game keeps the player's start values in the data table
`/Game/Sparta/Core/Characters/Player/Common/DT_PlayerAttributes` (row struct `AttributeMetaData`, an engine struct):
one row per gameplay attribute, named `<AttributeSet>.<Attribute>`, each with `BaseValue`, `MinValue` and `MaxValue`.
The player's character data `CD_Player` uses it, and so do the decoys (`CD_Decoy`, `CD_Decoy_Bait`, `CD_Decoy_Tiel`);
no enemy uses it. The mod changes two `BaseValue`s:

| Row | Game | Mod |
|---|---|---|
| `PlayerAttributeSet.HealAmountPerCharge` | 40 | 70 |
| `SpartaCombatSet.BaseDamageReduction` | 0 | 0.5 |

All other rows are the game's values of the version above.

- Heal: the heal charge effect `GE_HealingCharge` runs the game's heal calculation (`SpartaHealExecution`) with
  `BaseHeal` = `HealAmountPerCharge` x 1.0; `GE_HealingChargeMixed` gives half of it as health and half as Resolve. The
  healing charge upgrades (`GE_HealingChargeUpgrades`) multiply `HealAmountPerCharge`, so they scale from 70.
- Damage reduction: the game's damage calculation (C++) reads it, so its exact formula is not visible in the assets.
  The game uses it as a share of the damage taken: Harden (`GE_Harden_Active`) sets it to 1.0, and shell abilities,
  upgrades and items add to it (`GE_ShellDamageReduction`, `GE_FightStance_Active`, `GE_Eredrim_Tank`,
  `GE_PP_Trollweed_Passive`, `GE_PP_Moonshine_Passive`). So 0.5 is probably half the damage, and those bonuses add to
  it.

The mod replaces whole assets. A mod that changes the same assets conflicts with this one, and a game update that
changes them needs a new build of the mod.

## Build

Requirements:

- Windows PowerShell 5.1 (part of Windows).
- [UAssetGUI](https://github.com/atenfyr/UAssetGUI/releases) v1.1.0 and [retoc](https://github.com/trumank/retoc/releases)
  v0.1.5. `build.ps1` looks for `_tools\uassetgui\UAssetGUI.exe` and `_tools\retoc\retoc.exe` in the parent folders
  of the project; set `UE5_TOOLS` to a folder that holds both to use another place.
- The game's mapping file, `MortalShell2-1.12.0.usmap`, in `%LOCALAPPDATA%\UAssetGUI\Mappings\`.
- For the check of the build (optional): Python 3, the game installed on the same drive as the project, and, when
  the game's files are encrypted, its AES key in `_tools\gamedata\mortalshell2\game.json` (`{"aesKey": "0x..."}`).

```powershell
powershell -ExecutionPolicy Bypass -File .\build.ps1             # dist\mortalshell2_moredefandheal_P.pak/.utoc/.ucas
powershell -ExecutionPolicy Bypass -File .\build.ps1 -Deploy     # build, then copy the files to Paks\~mods
powershell -ExecutionPolicy Bypass -File .\deploy.ps1 -Wait      # copy the files when the game is closed
```

The build converts every asset source in `src` (UAssetGUI JSON) into a legacy `.uasset`/`.uexp` (UAssetGUI), packs
them into an IoStore container (retoc), and then checks the result: it converts the container back and compares
each asset with its source (`scripts\check_build.py`).

`deploy.ps1` copies the three files to `<game folder>\MortalShell2\Content\Paks\~mods`. The game folder is
`$env:UE5_GAMES_DIR\Mortal Shell II`; `-GameDir` or `-ModsDir` override it. The running game locks the files, so the
script stops (exit code 2) or, with `-Wait`, waits until the game is closed.

Versioning: every change raises the second number in `mod.json` (1.0 -> 1.1): `python scripts\bump_version.py`.

## Project layout

```
src/                               Mod source: the changed assets as UAssetGUI JSON, in the game's folder layout
dist/                              Build output: the .pak/.utoc/.ucas for Paks\~mods (git-ignored)
docs/                              Cover and screenshots for openmods.md
scripts/check_build.py             Compares the built assets with their sources
scripts/bump_version.py            Raises the version in mod.json
build.ps1                          Builds dist\ from src\ and checks the result
deploy.ps1                         Copies the build to the game's Paks\~mods folder
mod.json                           Name, version, game, engine and file names
```

The first commit with an asset holds the original asset, so `git diff` against it shows exactly what the mod
changes.

## License

MIT, see [LICENSE](LICENSE).
