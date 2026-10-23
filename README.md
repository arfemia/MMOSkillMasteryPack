# MMO Skill Mastery - Standalone Content Pack

A Hytale asset pack that ships the **mastery system and matching content** for
the [MMO Skill Tree plugin](https://wintergreen-solutions.com). The mastery tracks
are not bundled inside the plugin jar - this pack is the source of that content,
and the plugin hides the Mastery UI tab unless it's installed. The two wallets the
tracks are priced in, Mastery Points and Life Essence, and the Mastery Point ladder
come with MMO Skill Tree itself from 2.0.0.

## What's included

- **28 active mastery tracks** (`Server/MMOSkillTree/Masteries/*.json` +
  `Server/MMOSkillTree/MasteryGenerators/*.json`) - 11 combat-skill tracks,
  6 marquee ability tracks, 3 gathering tracks (Mining, Woodcutting,
  Harvesting), the Enchanting utility track, and 7 damage-school tracks (Fire, Ice, Lightning, Water,
  Arcane, Void, Poison) gated on combat level rather than one skill. Each
  track has 3-5 finite identity nodes plus one infinitely-repeatable
  "Eternal" node; the school tracks run 10. The per-skill families are
  written by three generator files; the schools, the marquee abilities,
  Magic and Defense are hand-authored files. The utility family (Enchanting,
  Acrobatics, Fishing) is written by one generator with a per-row on/off
  switch: `enchanting_mastery` is on; `acrobatics_mastery` and
  `fishing_mastery` ship with `"Enabled": false` and will activate in a
  future pack release.
- **No currency of its own.** The tracks are priced in `mastery_point` (counter-backed)
  and `life_essence` (item-backed, it wraps the Hytale `Ingredient_Life_Essence` item),
  and both wallets come with MMO Skill Tree itself. So does the Mastery Point ladder
  (one Mastery Point at every 15th level of every skill, 15 to 195, collected from the
  Mastery section of the achievement book, stamped as `<skill>_mastery_<level>`), which
  this pack's quests and achievements sit beside.
- **Mastery-themed quests + achievements** (`Server/ZiggfreedCommon/Quests/MMOSkillTree/Mastery/`,
  `Server/ZiggfreedCommon/Achievements/MMOSkillTree/Mastery/`) - content that uses the mastery /
  currency surface (e.g. "purchase your first mastery node", "complete a
  mastery track", "accumulate mastery points").

## Installation

Drop `MMOSkillMasteryPack-2.2.0.zip` (or this whole folder) into the same `mods/`
directory as the MMO Skill Tree plugin. Restart the server. Look for these
log lines:

```
[AssetPacks] Mastery asset layer applied (17 entries) - 15 masteries effective
[AssetPacks] Mastery generator layer applied (3 generators) - 28 masteries effective
[AssetPacks] Shared-schema achievement layer applied (N achievements) - N achievements effective
```

## Build (from source)

```powershell
.\build.ps1                  # build the zip, and install it if a Mods folder is known
.\build.ps1 -Install:$false  # build only, no copy
```

The script is self-contained and cross-platform (`pwsh ./build.ps1` works on macOS/Linux). It zips with the forward-slash plus directory entries Hytale needs; never use `Compress-Archive`. To auto-install on build, set `HYTALE_MODS_DIR` once to your Hytale `UserData/Mods` folder (or pass `-ModsDir <path>`); without it the script just builds the zip.

## Without this pack

The plugin still runs. XP, skill tree, skill rewards, quests, and achievements
all work normally. The differences:

- The **Mastery** menu tab and Mastery page are hidden
  (`MasteryConfig.isAvailable()` returns false on an empty track set).
- Both wallets, Mastery Points and Life Essence, still come with MMO Skill Tree itself,
  so the MMO's `Currency` feature (`MmoFeatures`) and the onboarding's currencies topic
  it gates stay on, and Mastery Points still wait in the achievement book every 15 levels
  of each skill, for the Techniques a trainer teaches.
- The `/mmomastery` admin command reports "no tracks configured".

## Authoring your own pack on top

The pack uses the standard MMO content-pack format (see CONTENT_PACKS.md in the
plugin repo). To extend or override what this pack ships:

- Ship your own `Server/MMOSkillTree/Masteries/<Id>.json` files to add tracks
  alongside these; a file with the SAME id as one of this pack's tracks
  replaces it (last pack loaded wins), and `"Enabled": false` on that id
  switches it off.
- Reuse this pack's `mastery_base` / `mastery_school_base` as your own
  `"Parent"`, or write a whole family with one
  `Server/MMOSkillTree/MasteryGenerators/<Id>.json` file.
- Server owners can always override anything authored by a pack with
  per-track entries in `mods/mmoskilltree/mastery.json` - owner-authored
  content wins over pack content, leaf by leaf.

## Sync rule

This pack is the source-of-truth for the mastery tracks and their quests and
achievements. The plugin ships no defaults for these types; if you change a track
here, the change reaches players when the pack is reloaded. The wallets and the
Mastery Point ladder are MMO Skill Tree's own: retune them with a same-named file
in your own pack, never by copying them into this one.
