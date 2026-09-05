# CLAUDE.md - MMOSkillMasteryPack

This directory is **a standalone Hytale content pack** that ships as its own
mod on CurseForge alongside the [MMOSkillTree mod](https://www.curseforge.com/hytale/mods/mmo-skill-tree).
The mod no longer ships `MasteryDefaults`/`CurrencyDefaults` in its jar; this
pack is the source of those defaults plus mastery-themed quests + achievements.
**Commerce content (the mastery-point conversion offer and both wallet
definitions) speaks ziggfreed-common's `zc-commerce` asset surface, not the
retired MMO `Currency`/`ShopEntry` types** - see "Commerce content
(wallets + the token-shop offer)" below.

## Layout

```
skill-mastery-pack/
├── manifest.json                              Hytale plugin manifest
├── CLAUDE.md                                  this file
├── CURSEFORGE.md                              CurseForge listing copy
├── README.md                                  end-user installation notes
└── MMOSkillMasteryPack-<version>.zip          built artifact (gitignored; build.ps1 takes the version from manifest.json)
└── Server/
    ├── Languages/<bcp47>/mmoskilltree.lang    display text, all nine locales
    ├── MMOSkillTree/
    │   ├── Control/MMOSkillMasteryPack.json   replace/add per content type (six keys, all "add": CommandRewards, Quests, Achievements and their three *Templates stores; Masteries and MasteryGenerators merge natively by id, no key)
    │   ├── Masteries/*.json                   2 Abstract bases (Mastery_Base, Mastery_School_Base) + 15 hand-authored tracks
    │   ├── MasteryGenerators/*.json           3 generators writing the other 15 tracks (combat / gathering / utility families)
    │   ├── CommandRewardTemplates/*.json      1 template (Mastery_Point_Milestones); see "CommandReward template extension" below
    │   ├── CommandRewards/MMOSkillMasteryPack.json   one {{ALL_SKILLS}} entry fans the template to every skill
    │   ├── QuestTemplates/*.json              (none shipped; Control already declares the key)
    │   ├── Quests/Pack_Mastery_Tithe.json     the ONE quest still on the old raw-Payload compat adapter (see "Quests + Achievements" below)
    │   └── AchievementTemplates/*.json        (none shipped; Control already declares the key)
    ├── Models/Mmo_Mastery_Trainer.json      the trainer's look (a Parent clone of Kweebec_Sapling, re-skinned and scaled)
    ├── NPC/Roles/Passive/Mmo_Mastery_Trainer.json   the trainer's role body (FULL body, not a variant: its press-F Hint is read literally)
    ├── Languages/<bcp47>/npcs.lang          the trainer's nameplate + press-F hint keys, all 9 locales
    ├── Item/Items/Weapon/Mmo_Adepts_Staff.json      the tribute's rarest pull (a Parent clone of Weapon_Staff_Bo_Wood, Epic, not craftable)
    ├── Languages/<bcp47>/items.lang         the staff's name + description keys, all 9 locales
    └── ZiggfreedCommon/
        ├── Quests/MMOSkillTree/Mastery/*.json           6 native quests (Pattern A, ziggfreed-common's QuestAsset)
        ├── Achievements/MMOSkillTree/Mastery/*.json     6 native achievements (Pattern A, ziggfreed-common's AchievementAsset)
        ├── Currencies/MMOSkillTree/Mastery_Point.json   the mastery-point wallet (counter-backed)
        ├── Currencies/MMOSkillTree/Life_Essence.json    the life-essence wallet (item-backed on Ingredient_Life_Essence)
        ├── Dialogues/MMOSkillTree/Mmo_Mastery_Trainer.json   the trainer's conversation; Start.First is the introduction beat, Start.Quests lists this pack's quests by id
        ├── NpcPlacements/Mmo_Mastery_Trainer_Temple.json     places the trainer in the Forgotten Temple
        ├── Lootables/Mmo_Mastery_Offering.json               what the daily tribute pays out (a level+luck ladder plus three bonus rolls)
        └── ShopEntries/MMOSkillTree/Shop_Convert_Mastery_Point.json   the token -> mastery-point conversion offer
```

## Release notes (patch-notes paradigm)

Per-version public release notes live in `patch-notes/<version>.md`, same paradigm as the main mod repo: YAML frontmatter (`version`, `title`, `type: patch-note`, `status: held|released`), a one-line summary, then user-facing `- **New/Fixed: ...**` bullets. No em-dashes. `patch-notes/_INDEX.md` lists them newest-first. `CURSEFORGE.md` is the public listing copy; keep its Versions table in sync with each release. (No docs-site publishing for packs yet.)

## Build & deploy

```powershell
.\build.ps1                  # build the zip, and install it if a Mods folder is known
.\build.ps1 -Install:$false  # build only, no copy
.\build.ps1 -ModsDir <path>  # build + install into an explicit folder
```

`build.ps1` is self-locating and cross-platform (Windows PowerShell, or `pwsh ./build.ps1` on macOS/Linux). It zips with the lower-level `[IO.Compression.ZipFile]` API using forward-slash entries plus an explicit directory entry per ancestor path; PowerShell's `Compress-Archive` writes backslash separators Hytale's asset loader silently drops, so never use it (see plugin-level memory for full details). To auto-install on build, set `HYTALE_MODS_DIR` once to your Hytale `UserData/Mods` folder (or pass `-ModsDir`); without it the script just builds the zip.

Top-level docs (`CLAUDE.md`, `CURSEFORGE.md`, `README.md`) are explicitly
excluded - they're for humans browsing the repo, not for Hytale to load.
The `.unused-for-now/` directory is also excluded (`$ExtraExcludeDirs` in `build.ps1`) - it holds mastery JSONs
that have been parked out of `Server/MMOSkillTree/Masteries/` and should
not ship to players until they're moved back. Keep edits there in sync
with the live tracks so an unpark is a straight move with no cleanup.

**Lighter-weight alternative: `"Enabled": false`** at the top of a track file
(or in a generator's `Child`, which is how the whole utility family ships)
parks it without moving the file. The track still decodes, so a server owner
can flip it back on from `mods/mmoskilltree/mastery.json` with
`{"tracks": {"<id>": {"Enabled": true}}}`. Use `.unused-for-now/` for tracks
you don't want shipping at all (keeps the zip leaner); use `Enabled` for
tracks you're toggling on and off during balance work.

## Pack JSON conventions

**Asset key comes from the filename**, which Hytale requires in PascalCase
(`Fireball_Mastery.json` is the track id `fireball_mastery`). Mastery tracks are
STRUCTURED files - PascalCase fields at the TOP level, no `Name`/`Payload`
wrapper:

```json
{ "Parent": "Mastery_Base",
  "Target": "skill:SWORDS",
  "Nodes": { "swo_t1_dmg": { "Tier": 1, "Modifiers": [ ... ] } } }
```

The two content types still on the older `Name`/`Payload` wrapper in this pack
are `CommandRewards`/`CommandRewardTemplates` and the one legacy quest
(`Quests/Pack_Mastery_Tithe.json`); their `Payload` is a nested JSON object the
plugin's own parser reads.

### Per-entry vs per-pack files

- **Mastery / Currency / Quest / Achievement** - per-entry; one file per
  track/currency/quest/achievement. The asset key (filename, lowercased)
  IS the runtime id for masteries and currencies. Legacy-shape quests +
  achievements get their runtime id from the inner Payload's `id` field, so
  that asset key doesn't have to match.
- **CommandRewards** - one file per pack. The Payload is the full
  `{ "<SKILL_ID>": { "<level>": [rewards…] } }` map.

### Inner id casing

Quest + achievement IDs in the inner Payload (`"id": "pack_first_milestone"`)
stay **lowercase** to match how the existing plugin defaults are written
and so cross-references (`metaChildren`, `prerequisites`, `currencyId`,
`MasteryCost.currencies`) remain consistent.

## Localization (display text)

All player-facing display text ships in `Server/Languages/en-US/mmoskilltree.lang` (loaded natively via `IncludesAssetPack: true`), keyed by convention. The mod's `LocalizedText` resolver tries an explicit key field, then the by-convention key, then a raw `displayName`/`description` (the deprecated fallback) - so pack JSON should set the key and leave the literal out:

- **Mastery tracks:** `mastery.<trackId>.title` (trackId = filename lowercased) and `mastery.<nodeId>.title` per node, resolved by convention; an explicit `TitleKey` leaf overrides the convention key. **Node DESCRIPTIONS auto-render - do NOT author a node `DescriptionKey` unless a node genuinely needs extra flavor.** The mastery page generates the effect line from the node's `Modifiers` (via `MasteryModifierRenderer`, localized through `mastery.mod.*`) and renders the cost separately as chips, so a hand-written description is dead duplication that also bakes un-localizable balance data. Author the modifier + cost + requirements as structured fields; the text follows.
- **Quests / bounties:** `quest.<id>.title` + `quest.<id>.flavor`, authored as `Text.TitleKey`/`Text.FlavorKey` on the native quests (id = filename) and as the inner `Payload.id` on `Pack_Mastery_Tithe.json` (the one still on the old schema).
- **Currencies:** a wallet's `Text.TitleKey` (unauthored falls to `currency.<id>.name` by convention, id = filename lowercased) for COUNTER-backed currencies (`mastery_point`). An ITEM-backed currency (`life_essence`) ships neither: with nothing authored, ziggfreed-common's naming ladder derives the display name from the backing item's native, already-translated lang key (`server.items.Ingredient_Life_Essence.name`, "Essence of Life"), exactly like the icon derives from the item - zero hand-maintained translations, and the currency can never disagree with the inventory tooltip.
- **Achievements:** `achievement.<id>.title` + `achievement.<id>.desc`.
- **Shop offers:** `Text.TitleKey`/`Text.FlavorKey` on the offer JSON pointing at `.lang` keys.

Translate by shipping `Server/Languages/<bcp47>/mmoskilltree.lang` with the same keys (missing keys fall back to English per key). Reward line-items carry NO `displayName`: the mod auto-renders a localized line from the reward itself ("+5 Mastery Points", "Taming XP x2 for 45m", native item names), so a baked-in literal would double-render the amount AND pin the text to English. No em-dashes in `.lang` values.

## Mastery authoring (structured schema, plugin 1.6.0+)

A track is one structured file at `Server/MMOSkillTree/Masteries/<Id>.json` - PascalCase
fields at the top level, decoded by the plugin's `MasteryAsset` codec (the codec IS the
schema; the full field reference is the plugin's `CONTENT_PACKS.md` "Mastery tracks"
section). Reuse comes in two shapes, both native:

**1. `Parent` against an `Abstract` base.** `Mastery_Base.json` (refund policy) and
`Mastery_School_Base.json` (global placement + the combat-level entry gate) are the two
shipped bases; every track names one with `"Parent": "<Exact_Filename>"` and inherits per
LEAF - a child retunes one node's one price and keeps everything else, because `Nodes`
merges per node id and every group inside a node merges per field. `Abstract` itself never
inherits, so a child of a base is a real track.

**Casing: `Parent` and a generator's `Base` spell the same target differently.** A `Parent`
is the target file's name minus `.json`, exactly as written (`"Mastery_Base"`), because the
engine resolves the chain against the raw asset key before the mod ever folds it. A
generator's `Base` is looked up after that fold, against ids the mod has lower-cased, so it
is written `"mastery_base"` - and a `Base` that misses reports an `UNKNOWN_BASE` finding
naming the member it skipped.

**2. A generator for a whole family.** `Server/MMOSkillTree/MasteryGenerators/<Id>.json`
writes "the same track, once per skill" from one file:

```json
{ "Base": "mastery_base",
  "IdPattern": "{skillLower}_mastery",
  "ForEach": [ { "Values": [
      { "skill": "SWORDS", "skillLower": "swords", "prefix": "swo",
        "dmgIngredient": "Ingredient_Lightning_Essence", "sacrificeStat": "HEALTH" } ] } ],
  "Child": { "Target": "skill:{skill}",
             "Nodes": { "{prefix}_t1_dmg": { "Modifiers": [
                 { "Shape": "PERCENT", "ParamKey": "damage", "Value": 0.08,
                   "TargetSkill": "{skill}", "CombatTarget": "{skill}" } ] } } } }
```

Substitution applies to every string value, every object KEY (that is how each member
gets its own `{prefix}_*` node ids), and `IdPattern`; a value that is exactly one token
keeps its type, so a count lands as a number. Each combination becomes an ordinary child
of `Base`, decoded exactly like a hand-written file - and an authored `Masteries/` file
of the same id always WINS over the generated member, which is how one member of a
family is retuned without leaving it. A member that diverges too far is simpler as a
hand file the generator never writes: Magic and Defense are omitted from the combat
generator's `ForEach` and authored whole (Magic carries an extra L100 capstone node,
Defense points its nodes at Shield Slam and Guardian's Call).

The three shipped generators: `Combat_Skill_Masteries` (the six-node combat shape, once
per weapon skill; per-row tokens carry each skill's ingredients, sacrifice stat, and
Archery's dearer opener), `Gathering_Skill_Masteries` (the four-step loot-luck shape,
level-gated), and `Utility_Skill_Masteries` (the same shape with no level gates past
the entry one, whole family shipped parked with `Enabled: false`).

Node ids are a stability contract: a player's purchases are saved under
`<trackId>:<nodeId>`, so renaming either orphans everybody who bought the node.

## Damage-school mastery tracks (plugin 1.6.0+)

Seven hand-authored tracks (`{Fire,Ice,Lightning,Water,Arcane,Void,Poison}_School_Mastery.json`,
each `"Parent": "Mastery_School_Base"`) buy into a DAMAGE SCHOOL rather than a skill or an
ability. The plugin's `School` modifier scope is orthogonal to every other scope
and outranks all of them: an authored `"School"` sends the value to that school's
own stat channel, so it pays out on every hit of that school no matter which
weapon, skill, or ability produced it. Two param keys accept it and no others:

```json
{ "Shape": "PERCENT", "ParamKey": "damage",       "School": "Fire", "Value": 0.05 }
{ "Shape": "FLAT",    "ParamKey": "damage",       "School": "Fire", "Value": 8    }
{ "Shape": "PERCENT", "ParamKey": "schoolResist", "School": "Fire", "Value": 0.04 }
```

`PERCENT` values are fractions (0.05 is +5%), `FLAT` is raw damage, and school ids
are matched case-insensitively. **`schoolResist` is PERCENT only** (its channel
already counts whole percent points, so a FLAT 4 and a PERCENT 0.04 would be one
bonus written two ways) and a school may never sit beside `TargetAbility` /
`TargetSkill` / `CombatTarget`, nor combined with a `Condition` - each is a load-time
error naming the fix.

**The school base carries `"Target": "global"`, so every school track inherits it.**
A track's target is its own
placement width, and the three widths are `ability:<id>`, `skill:<ID>` and the bare
word `global`. A skill-scoped track only ever emits its per-ability modifiers for an
ability that routes XP to that skill, so a Fire track parked on `skill:MAGIC` would
silently drop its lines for the Blunt slam and the Archery shot that also burn.
`global` reaches every ability, which is the whole point of a school: what decides a
value is the hit's damage school, never which skill threw it. The school spine is
unaffected either way (a `School`-scoped value rides its own channel), and an
unscoped `damage` value on a global track lands on the global damage channel.

Shape of a track, every node gated on `MMO_CombatLevel` (16 track / 40 T2 / 55 F1 /
70 T3 / 76 F2 / 80 T4 / 88 F3 / 92 Eternal) so one track suits a mage and a rogue
alike:

| Node | What it buys |
|------|--------------|
| `<prefix>_t1_pct` / `<prefix>_t1_flat` | choice pair: percent vs flat school damage |
| `<prefix>_t2_resist` | school resistance, open from either opener |
| `<prefix>_f1_focus` | flavor: one ability of the school, sharpened (behind T2) |
| `<prefix>_t3_cap` | capstone, bigger percent + a permanent max-stat sacrifice (behind T2) |
| `<prefix>_t4_cd` / `<prefix>_t4_flat` | Ascendant choice pair: school-wide cooldown vs more flat damage |
| `<prefix>_f2_finesse` | flavor: a signature effect made to last (behind the capstone) |
| `<prefix>_f3_secret` | flavor: the school's parting trick (behind `f2_finesse`, the deepest node) |
| `<prefix>_eternal` | repeatable +1% school damage, EXPONENTIAL 1.1 |

The spine is school-channel damage and resistance; the three flavor nodes are the
per-ability half, and they sit off the spine behind a prerequisite rather than in
the opening tier, so a player finds them by climbing rather than by reading the
first screen.

Each school file carries all ten nodes in full: the spine slots are the same shape
per school, while the `t4_cd` sweep and the three flavor nodes point at that
school's own abilities (`f1_focus` cuts a cooldown or a cost, `f2_finesse`
stretches a duration, `f3_secret` widens a radius or a reach). Point a
per-ability modifier only at a `Key` the ability's own Body declares:
`durationMs`, `radius`, `damageRadius`, `distance`, `pierceCount` are body params,
`cooldownMs` and `cost` are ability-level and work on any ability that has one.
A key an ability never declares is inert rather than an error (the `mastery` audit
domain under `/mmoconfig validate` warns on one outside the canonical vocabulary),
so check the ability JSON before picking one - `fireball` declares `damageRadius`,
not `radius`, and a passive capstone with no `Cooldown` block has no `cooldownMs`
to cut.

Titles: track titles are `mastery.<trackId>.title`, the shared tier names are
`mastery.node.<tail>.title` (`t1_pct`, `t1_flat`, `t2_resist`, `f1_focus`,
`f2_finesse`, `f3_secret`, `t4_cd`, `t4_flat`, composed as "`<track>: <tier>`"),
and each capstone gets its own per-node `mastery.<prefix>_t3_cap.title` so it can
carry a school-flavored name. Node descriptions still auto-render from the
modifiers; do not author one.

## CommandReward template extension (plugin 1.1.0+)

Pack-shipped CommandRewards can extend reusable per-skill-block templates.
Templates live in `Server/MMOSkillTree/CommandRewardTemplates/*.json` and
load before CommandRewards. A template Payload is a level&rarr;rewards map
(same shape a single skill block carries) optionally with `{{paramName}}`
substitution tokens:

The template's filename (lowercased) is the id a block `extends`, so the
underscores in `Mastery_Point_Milestones.json` MUST match the `extends` value
`mastery_point_milestones` (the resolver lowercases but does NOT insert
underscores at case boundaries; a `MasteryPointMilestones.json` file would
resolve to id `masterypointmilestones` and never match).

```json
{
  "Name": "Mastery_Point_Milestones",
  "Payload": {
    "15": [{ "command": "/mmocurrency give --player={player} --currency=mastery_point --amount=1", ... }],
    "30": [{ "command": "...", ... }],
    ...
  }
}
```

A CommandRewards Payload then uses per-skill `extends` references. The
**`{{ALL_SKILLS}}` sentinel** as a top-level key fans the template out to
every known skill that isn't otherwise explicitly listed in the same Payload:

```json
{
  "Name": "MMOSkillMasteryPack",
  "Payload": {
    "{{ALL_SKILLS}}": { "extends": "mastery_point_milestones" }
  }
}
```

DSL per skill block (resolved by `CommandRewardTemplateResolver`):

1. **`extends: "<template-id>"`** - pull the template Payload (case-insensitive id lookup).
2. **`params: { ... }`** - feed `{{paramName}}` substitution. Empty resolve drops the holding key.
3. **`levelOverrides: { "<level>": [rewards] }`** - replace a level's reward
   array wholesale. The level must exist in the template - unknown levels
   warn-and-skip (use `extraLevels` for new ones).
4. **`extraLevels: { "<level>": [rewards] }`** - add new levels. Collision
   with a template level warns-and-skips (use `levelOverrides` to modify).
5. Any non-reserved top-level field overlays the template.

The sentinel only fans out to skill keys - `TOTAL` and `GLOBAL_SKILL`
(the special non-skill keys) are never touched. The mastery pack's
CommandRewards file collapsed from 2,386 lines (240 identical entries) to
6 lines once the template + sentinel landed.

The pack `Control/MMOSkillMasteryPack.json` adds `"CommandRewardTemplates": "add"`.

## Quest + Achievement templates (plugin 1.1.0+)

Quests and achievements also support top-level `extends` against per-type
template stores. The DSL mirrors Mastery's, with field names that match
each content type's primary collection:

| Content type   | Template directory                | Override field         | Append field        | Array field    |
|----------------|-----------------------------------|------------------------|---------------------|----------------|
| Quest          | `QuestTemplates/*.json`           | `objectiveOverrides`   | `extraObjectives`   | `objectives`   |
| Achievement    | `AchievementTemplates/*.json`     | `criterionOverrides`   | `extraCriteria`     | `criteria`     |

Example quest using a template:

```json
{
  "Name": "Pack_Daily_Kill_Goblin_T1",
  "Payload": {
    "extends": "daily_kill_template",
    "params": { "id": "pack_daily_kill_goblin_t1", "displayName": "Goblin Hunter I",
                "mobId": "Mob_Goblin_T1", "count": "10", "reward": "5" },
    "objectiveOverrides": {
      "kill_target": { "displayText": "Slay 10 Goblins" }
    }
  }
}
```

Same resolution pipeline as the other resolvers:
1. Deep-clone template Payload.
2. `{{paramName}}` substitution (empty resolve drops the holding key).
3. Non-reserved track-level keys overlay the template (id, displayName,
   description, rewards array as a whole, etc. - so the easiest way to
   replace rewards is to supply a fresh `rewards: [...]`).
4. `*Overrides` deep-merge into the matching array entry by `id`.
5. `extra*` append new entries (collision with template id is rejected).

The pack `Control/MMOSkillMasteryPack.json` adds `"QuestTemplates": "add"`
and `"AchievementTemplates": "add"` alongside the existing content-type modes.

The pack ships no `QuestTemplates/` or `AchievementTemplates/` folder today (its
`Control/` file already declares both keys, so creating one is enough). This DSL only
applies to `Pack_Mastery_Tithe.json`, the one quest still on the old raw-Payload compat
adapter - see "Quests + Achievements (native Pattern A)" below for where the rest of the
pack's quest/achievement content actually lives now.

## Quests + Achievements (native Pattern A, 1.6.0+)

Every quest and achievement in this pack EXCEPT `Pack_Mastery_Tithe.json`
authors directly against ziggfreed-common's own `QuestAsset` /
`AchievementAsset` codecs (Pattern A: the codec IS the schema, no `Name`/
`Payload` wrapper, no `extends`/`params` DSL) at:

```
Server/ZiggfreedCommon/Quests/MMOSkillTree/Mastery/*.json
Server/ZiggfreedCommon/Achievements/MMOSkillTree/Mastery/*.json
```

Three shapes every file here uses:

- **`Objectives` / `Criteria` are id-keyed maps**, not arrays: `"Objectives": {"reach_l15":
  {"Kind": "REACH_LEVEL", "Target": "ANY", "Amount": 15}}`. The key is what a player's
  progress is filed under, so renaming one restarts that step and the key is as much a
  stability contract as the file's own id.
- **`Rewards` is an envelope**, `{"Claim": [...]}` or `{"Auto": [...]}`. `Claim` is what
  every file here ships: the player collects at the trainer rather than being paid the
  instant the last step ticks over.
- **`Npc: {"ViewId": "Mmo_Mastery_Trainer", "TurnInId": "giver"}`** is what puts a quest in
  the trainer's list and takes its hand-in there. Add the quest id to the trainer
  dialogue's `Start.Quests` too, so a player returning with it finished is greeted with the
  hand-in ahead of the menu (see "The Mastery Trainer" below).

**The FILENAME is the id** (lower-cased at decode; the `MMOSkillTree`/
`Mastery` folders are pure organization and contribute nothing to the id,
per ziggfreed-common's `_`-marked-folder convention - only a folder whose
name STARTS with `_` prefixes the id). Renaming one of these files renames
its id, and quest/achievement ids are a stability contract (dialogue
bindings, prerequisites, and a player's saved progress all bind by exact
id) - never rename one casually. `Pack_Essence_Hoarder_10000.json` is a
worked example of this: its filename had to change from the pre-1.6.0
`Pack_Essence_Hoarder_1000.json` to match its STABLE inner id
(`pack_essence_hoarder_10000`), which the old `Name`/`Payload` shape let
drift from the filename; the new shape can't have that drift because there
is no separate id field to hide behind.

No `Control/` entry is needed for these two folders: ziggfreed-common's own
`FrameworkAssetRegistrar` registers the `Quests`/`Achievements` stores once,
process-wide, and pack-level overrides are native override-by-id
(ship a same-named file, or `"Enabled": false` to retire one) rather than a
`PackControlAsset` mode.

A file here carries no attribution leaf: the shared store is the server's,
and every reader folds all of it, so what keeps this content out of a server
that cannot use it is its gate rather than a label. `Requires` gates on the
`mmoskilltree:feature` factor: `{"Factors": [{"Factor":
"mmoskilltree:feature", "Param": "mastery", "Min": 1}]}` keeps the content
out of circulation entirely while the mastery feature is off.

A skill-level objective or criterion is written as `{"Kind": "REACH_LEVEL",
"Target": "<SKILL>"}` - `TOTAL` for the summed level and `ANY` for the
highest single skill. It desugars onto the raw `STAT_THRESHOLD` stat-channel
form on the way into the engine, so the raw form (`"Target":
"MMO_Level_<SKILL>"` / `MMO_HighestSkillLevel` / `MMO_TotalLevel`) works too
and is the way to write a step about a number this mod does not own. Both
render identically; prefer the friendly one, which is what the jar's own
content uses.

A tiered achievement declares its ladder on the shared
`Listing.Chains: [{"Id": "<ladder>", "Tier": n}]` leaf, and a description
key with a `{0}` in it gets its number from `Text.TextArgs.Flavor:
["@amount"]` rather than from a per-rung translation.

**Why `Pack_Mastery_Tithe.json` is still on the compat adapter**: nothing in
its shape blocks a conversion. Its `requiredAchievements:
["pack_mastery_hoarder_300"]` gate has a native spelling - the shared
`ziggfreedcommon:achievement_earned` factor: `{"Factors": [{"Factor":
"ziggfreedcommon:achievement_earned", "Param": "pack_mastery_hoarder_300",
"Min": 1}]}` beside the `mmoskilltree:feature` factor the other files
already carry. Its `repeatable`/`cooldownSeconds` become the `Repeat`
block's `Cooldown`, `autoAccept` becomes `Flow.AutoAccept`, and the
`minLevel` requirement becomes an ordinary factor bound on the total-level
channel. Converting it is a straight rewrite into
`Server/ZiggfreedCommon/Quests/MMOSkillTree/Mastery/`, and the file stays
here only until somebody does it.

## The Mastery Trainer (NPC placement + dialogue)

**This pack owns the trainer end to end.** The mod jar ships nothing about that character:
their look (`Server/Models/Mmo_Mastery_Trainer.json`), their role body
(`Server/NPC/Roles/Passive/Mmo_Mastery_Trainer.json`), their nameplate and press-F hint
(`Server/Languages/<bcp47>/npcs.lang`), their placement
(`Server/ZiggfreedCommon/NpcPlacements/Mmo_Mastery_Trainer_Temple.json`, role
`Mmo_Mastery_Trainer`, `Where.GameplayConfig: ["ForgottenTemple"]`, gated on the
`mmoskilltree:feature` / `mastery` factor) and their conversation all live here, so a
server without this pack has no trainer at all. The role is a FULL body rather than a
`Variant` of the jar's `Template_Mmo_Hub` because the engine reads a `SetInteractable`
`Hint` literally and the trainer's prompt says train rather than talk.

The placement's `Interact` names a `Dialogue`, never an `Open`: pressing F opens
`Server/ZiggfreedCommon/Dialogues/MMOSkillTree/Mmo_Mastery_Trainer.json`, which routes on
to the mastery screen from its own menu. Author one or the other, never both.

`Start.First` is the introduction: while `meet_the_mastery_trainer` is active the trainer
greets the player with a line written for a first meeting, and the option under it carries
the `MarkTalked` beat that CREDITS the talk step. That quest is the one exception to the
rule below - the Adventurer's Guide in the temple is its giver, so the player is SENT here
rather than arriving to find it waiting, and it finishes on this conversation with no walk
back.

Pressing F credits nothing on its own, so a talk objective aimed at this character always
needs that beat; move the beat if you re-point the step. Once the quest is finished the
line retires itself.

`Start.Quests` names this pack's mastery quests by id, so a player returning with one
finished is greeted with its hand-in beat ahead of the menu. Add a row there when you add
a quest that names the trainer as its giver, or its hand-in never gets that first-beat
treatment. Its lines are keyed `dialogue.mmo_mastery_trainer.*` in the pack's `.lang`
files.

Delete the PLACEMENT file (or set `"Enabled": false` for it in
`mods/ziggfreedcommon/npc-placements.json`) to fall back to the jar's placement and the
bare mastery screen; deleting only the dialogue leaves the placement pointing at a
conversation that is not there.

## The daily tribute (quest + lootable + the staff)

`Mastery_Tribute.json` is the trainer's repeatable daily; what it pays comes from
`Server/ZiggfreedCommon/Lootables/Mmo_Mastery_Offering.json`, not from the quest file. Every roll
there evaluates independently, so a lucky claim hands over several at once. The first roll is a
`Ladder`: combat level and luck are summed into one score and the highest `Floors` entry reached
wins, which is what makes the daily grow with the player. Both channels are read as WHOLE NUMBERS
(MMO_Luck stores whole percent points), so the floors sit on a scale of roughly 0 to 200; never
compose MMO_Luck with the `mmoskilltree:station_luck` aggregate, which returns a fraction and counts
the same investment twice. The other three rolls are a 40% mastery point, a 10% boost token and a
3% `Mmo_Adepts_Staff`; the last two carry `"Cue": "Rare_Find"`, the FeedbackMoment the MMO jar ships
as a toast and a chime. `Mmo_Adepts_Staff` is a `Parent` clone of `Weapon_Staff_Bo_Wood` that does
not inherit a Recipe, so the tribute table is the only way to get one; its stats live in
`Utility.StatModifiers` and its display text in `Server/Languages/<bcp47>/items.lang`. To change how
often the staff appears, edit its Roll's `Chance` in the lootable rather than the item.

## Commerce content (wallets + the token-shop offer)

Both wallets and the mastery-point conversion offer speak
ziggfreed-common's shared `zc-commerce` asset surface, the same schema the
bounty-contracts pack's boards and storefronts use (Pattern A, native
`Parent` inheritance, no `Name`/`Payload` wrapper and no `extends`/`params`
DSL):

```
Server/ZiggfreedCommon/Currencies/MMOSkillTree/Mastery_Point.json
Server/ZiggfreedCommon/Currencies/MMOSkillTree/Life_Essence.json
Server/ZiggfreedCommon/ShopEntries/MMOSkillTree/Shop_Convert_Mastery_Point.json
```

**This pack defines BOTH wallets itself, on purpose, so it stays fully
standalone.** `Life_Essence` is shipped identically here and in the
bounty-contracts pack; ziggfreed-common's keyed fold merges same-id files
from whichever packs are installed, so running one, the other, or both
never produces two disagreeing definitions. `Mastery_Point` is unique to
this pack, since nothing else grants it.

**The conversion offer renders in the General storefront the
bounty-contracts pack ships** (`Shop: "General"`, `Listing.Category:
"conversion"`), which makes the bounty-contracts pack a SOFT dependency for
that one offer: install this pack alone and the offer still exists and
still pays out, it simply has no storefront to appear on until a General
shop is present. Nothing else in this pack needs the bounty pack.

**The `Currency` reward kind replaced the retired MMO `Mmo_Currency`
kind** everywhere a quest, achievement or offer pays out mastery points:
`{"Kind": "Currency", "Params": {"Currency": "mastery_point", "Amount":
N}}` (the amount is written as a bare number; quotes also work). It is
unprefixed because ziggfreed-common owns the wallet engine
behind it; no `Presentation`/`NameKey` is authored on the reward entry
itself; the chip a player sees comes from the wallet's own file (an
authored `Text.TitleKey`, else the item-backed wallet's own name) via a
three-rung naming ladder, so a currency's display name lives in exactly
one place no matter how many rewards pay it out.

**`Shop_Convert_Mastery_Point.json` keeps its legacy id on purpose.** The
pre-1.3.0 offer's true id was `shop_convert_mastery_point` (an inner
`Payload.id` the old schema let diverge from the `Convert_Mastery_Point`
filename); a player's daily-purchase count is filed under that id, and
the new schema derives its id from the FILENAME alone, so the file is
named to match rather than mirroring the old filename. Do not rename it.

**Currency knobs the old MMO schema had that `CurrencyAsset` has no direct
leaf for** (`showOnSidebar`, `showOnMasteryPage`, `xpConversionPercent`)
travel through the wallet's namespaced `Meta.mmoskilltree` block instead of
being dropped, mirroring the bounty pack's own `Bounty_Token`/`Life_Essence`
files: `{"Meta": {"mmoskilltree": {"ShowOnSidebar": true, "ShowOnMasteryPage":
true, "XpConversionPercent": 0}}}`. The commerce engine itself never reads
this block; whether the mod's own UI still honors it is that side's concern.

## Editing the pack content

- **Mastery generators** (`Server/MMOSkillTree/MasteryGenerators/*.json`) -
  see "Mastery authoring" above. Edit a generator's `Child` to change the
  shared shape across its whole family; edit a `ForEach` row to retune one
  member's tokens; author a `Masteries/` file of a member's id to take that
  member out of the family entirely (an authored file always wins).
- **Mastery tracks** (`Server/MMOSkillTree/Masteries/*.json`) - one structured
  file per track, PascalCase, track id = filename (lowercased). `Target`
  (skill:X, ability:X, or global), optional `Requires`, and a `Nodes` map
  keyed by node id: each node carries `Tier`, `Cost` (`Currencies` +
  `Items` + `Combine`), optional `StatSacrifice`, `Modifiers`, and for
  repeatable Eternals `MaxPurchases: -1` + `CostScaling`. Full field
  reference: the plugin's `CONTENT_PACKS.md` "Mastery tracks" section.
  Reuse via `"Parent"` against one of the two `Abstract` bases.
- **Wallets + the token-shop offer** - see "Commerce content (wallets + the
  token-shop offer)" above for where this pack's currency and shop content
  actually lives now (`Server/ZiggfreedCommon/Currencies/` and
  `/ShopEntries/`, the shared zc-commerce schema).
- **Quests + Achievements** - see "Quests + Achievements (native Pattern A,
  1.6.0+)" above for where this pack's content actually lives. Only
  `Pack_Mastery_Tithe.json` still uses the old raw-Payload schema (same
  shape as the plugin's owner files under `mods/mmoskilltree/quests/`):
  reward shapes `CURRENCY` (`currency` + `amount`; the older `currencyId`
  spelling still parses), `XP` (`skill` + `amount`), `COMMAND` (`command`
  string), `BOOST_TOKEN`.

### Currency-grant command format

Always emit `/mmocurrency …` commands in the named-arg form the Hytale
parser expects. The canonical wire format is built by
`MmoCurrencyCommand.buildGiveCommand(player, currencyId, amount)`  - 
single source of truth for any template that needs to grant currency.

Correct:

    /mmocurrency give --player={player} --currency=mastery_point --amount=1

Wrong (legacy positional - the live command rejects it with
"Expected: 1, actual: 4" and silently grants nothing):

    /mmocurrency give {player} mastery_point 1

## Sync with plugin

The pack and the plugin co-evolve:

- If the plugin changes the mastery track schema (a `MasteryAsset` codec
  change), bump `MasteryConfig.SCHEMA_VERSION` there and re-emit the
  affected mastery JSON here.
- If the plugin adds a new content type, add a structured Pattern A asset
  class, register it in `AssetStoreRegistrar`, and you can immediately ship
  content of that type from this folder (no `PackControlAsset` key - the
  store merges by id natively).

## Verification

1. Build the plugin: `./gradlew build` from the monorepo root, two levels up (`../../`). Produces
   `build/libs/MMOSkillTree-X.Y.Z.jar`.
2. Build the pack zip (see Build & deploy above).
3. Copy both into your Hytale mods folder
   (`D:\Games\Hytale\UserData\Mods\`).
4. Start the server. Confirm in the server log
   (`Saves/<world>/logs/<date>_server.log`):
   - `[AssetPacks] Mastery asset layer applied (17 entries) - 15 masteries effective`
     then `[AssetPacks] Mastery generator layer applied (3 generators) - 27 masteries effective`
     (17 files = 2 bases + 15 tracks; the generators write the other 15, and the two
     bases plus the parked utility trio never count as effective).
   - Same for the CommandRewards, Quest and Achievement layers. There is no Currency
     line: the wallets are shared-economy assets under
     `Server/ZiggfreedCommon/Currencies/`, so a `Server/MMOSkillTree/Currencies/`
     line in the log means a file was left behind in the retired folder.
   - No `Failed to decode asset:` or `Asset validation FAILED` lines.
5. In-game: open the Mastery menu tab (should be visible only if the pack
   loaded), buy a node, confirm currency deducted. Level a skill past 15,
   confirm the mastery-point reward shows as claimable.
