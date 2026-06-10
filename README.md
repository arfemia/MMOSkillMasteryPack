# MMO Skill Mastery — Standalone Content Pack

A Hytale asset pack that ships the **mastery system, currencies, and matching
content** for the [MMO Skill Tree plugin](https://wintergreen-solutions.com). As
of plugin version 1.2.0 the mastery + currency systems are no longer bundled
inside the plugin jar — this pack is the source of that content, and the plugin
hides the Mastery and Currency UI tabs unless it's installed.

## What's included

- **20 active mastery tracks** (`Server/MMOSkillTree/Masteries/*.json`) — 11
  combat-skill tracks, 6 marquee ability tracks, 3 gathering tracks (Mining,
  Woodcutting, Harvesting). Each track has 3-5 finite identity nodes plus one
  infinitely-repeatable "Eternal" node. Three additional tracks
  (`fishing_mastery`, `enchanting_mastery`, `acrobatics_mastery`) ship with
  `"disabled": true` and will activate in a future pack release.
- **2 currencies** (`Server/MMOSkillTree/Currencies/*.json`) — `mastery_point`
  (counter-backed, paced by quest/milestone rewards) and `life_essence`
  (item-backed, wraps the Hytale `Ingredient_Life_Essence` item).
- **Mastery-point milestone rewards** (`Server/MMOSkillTree/CommandRewards/MMOSkillMasteryPack.json`
  + `Server/MMOSkillTree/CommandRewardTemplates/MasteryPointMilestones.json`)
  — +1 mastery_point every 15 levels for every built-in skill. Authored via the
  new CommandReward template system: one template + a single `{{ALL_SKILLS}}`
  entry fans out to every skill (vs. 2,386 lines of duplicated reward objects).
- **Mastery-themed quests + achievements** (`Server/MMOSkillTree/Quests/`,
  `Server/MMOSkillTree/Achievements/`) — content that uses the mastery /
  currency surface (e.g. "purchase your first mastery node", "complete a
  mastery track", "accumulate mastery points").

## Installation

Drop `MMOSkillMasteryPack.zip` (or this whole folder) into the same `mods/`
directory as the MMO Skill Tree plugin. Restart the server. Look for these
log lines:

```
[AssetPacks] Mastery pack layer applied (23 entries, mode=add) — 20 masteries effective (3 disabled)
[AssetPacks] Currency pack layer applied (2 entries, mode=add) — 2 currencies effective
[AssetPacks] CommandRewards pack layer applied (1 packs, mode=add) — N skill+level entries effective
[AssetPacks] Quest pack layer applied (5 entries, mode=add) — 5 quests effective
[AssetPacks] Achievement pack layer applied (6 entries, mode=add) — 6 achievements effective
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
- The **Currency** sidebar / page is hidden (`CurrencyConfig.isAvailable()`).
- No `mastery_point` rewards fire on level-up.
- The `/mmocurrency` and `/mmomastery` admin commands report "no currencies /
  no tracks configured".

## Authoring your own pack on top

The pack uses the standard MMO content-pack format (see CONTENT_PACKS.md in the
plugin repo). To extend or override what this pack ships:

- Create your own pack with `"Mastery": "add"` (default) in
  `Control/<yourpack>.json` to add tracks alongside these.
- Use `"Mastery": "replace"` to drop these defaults entirely and ship your own.
- Server owners can always override anything authored by a pack with files in
  `mods/mmoskilltree/` — owner-authored content wins over pack content.

## Sync rule

This pack is the source-of-truth for the mastery / currency content. The plugin
no longer ships defaults for these types; if you change a track or currency
here, the change reaches players when the pack is reloaded.
