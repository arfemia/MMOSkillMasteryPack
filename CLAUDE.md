# CLAUDE.md — MMOSkillMasteryPack

This directory is **a standalone Hytale content pack** that ships as its own
mod on CurseForge alongside the [MMOSkillTree mod](https://www.curseforge.com/hytale/mods/mmo-skill-tree).
The mod no longer ships `MasteryDefaults`/`CurrencyDefaults` in its jar; this
pack is the source of those defaults plus mastery-themed quests + achievements.

## Layout

```
skill-mastery-pack/
├── manifest.json                              Hytale plugin manifest
├── CLAUDE.md                                  this file
├── CURSEFORGE.md                              CurseForge listing copy
├── README.md                                  end-user installation notes
├── MMOSkillMasteryPack.zip                    built artifact (gitignored if you regenerate)
└── Server/
    └── MMOSkillTree/
        ├── Control/MMOSkillMasteryPack.json   replace/add per content type
        ├── MasteryTemplates/*.json            1 template (Combat6_Standard); see "Mastery template extension" below
        ├── Masteries/*.json                   20 mastery tracks (11 thin extends-files + 9 hand-authored)
        ├── Currencies/*.json                  2 currencies
        ├── CommandRewardTemplates/*.json      1 template (MasteryPointMilestones); see "CommandReward template extension" below
        ├── CommandRewards/MMOSkillMasteryPack.json   one {{ALL_SKILLS}} entry fans the template to every skill
        ├── QuestTemplates/*.json              (optional) reusable quest skeletons; see "Quest + Achievement templates" below
        ├── Quests/*.json                      5 hand-authored quests (incl. Pack_Mastery_Tithe — repeatable 6h claim)
        ├── AchievementTemplates/*.json        (optional) reusable achievement skeletons
        └── Achievements/*.json                5 hand-authored achievements
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
excluded — they're for humans browsing the repo, not for Hytale to load.
The `.unused-for-now/` directory is also excluded (`$ExtraExcludeDirs` in `build.ps1`) — it holds mastery JSONs
that have been parked out of `Server/MMOSkillTree/Masteries/` and should
not ship to players until they're moved back. Keep edits there in sync
with the live tracks (formatting, displayName conventions, etc.) so an
unpark is a straight move with no cleanup.

**Lighter-weight alternative:** add `"disabled": true` at the top of a
mastery file's `Payload` to park it without moving the file. The track
JSON still ships in the zip but `MasteryConfig.parseTrack` drops it
during the pack-layer apply, logging `MasteryConfig: skipping disabled
mastery track '<id>'`. Honored on base defaults, pack entries, and user
overrides identically. Use `.unused-for-now/` for tracks you don't want
shipping at all (keeps the zip leaner); use `disabled` for tracks you're
toggling on and off during balance work.

## Pack JSON conventions

**Asset key comes from the filename**, which Hytale requires in PascalCase
(`Swords_Mastery.json`, not `swords_mastery.json`). The `Name` field inside
each JSON file is a human-readable echo of the asset key — the codec
(`AbstractRawJsonAsset.rawCodec`) consumes it via a no-op setter so it
doesn't trip the "Unused key(s)" warning, and echoes it on encode for
round-trip via `/mmopacks export`.

```json
{
  "Name": "Swords_Mastery",
  "Payload": { "target": "skill:SWORDS", "displayName": "Swords Mastery", … }
}
```

`Payload` is a **nested JSON object** (not an escaped string). The codec
captures it via `Codec.BSON_DOCUMENT`; the merge handler converts to Gson
`JsonObject` via `bson.toJson(JsonMode.RELAXED)` and feeds it to the
existing config parser (`MasteryConfig.parseTrack`, `QuestConfig.parseQuest`,
etc.).

### Per-entry vs per-pack files

- **Mastery / Currency / Quest / Achievement** — per-entry; one file per
  track/currency/quest/achievement. The asset key (filename) becomes the
  runtime id for masteries and currencies (currencies lowercased; masteries
  lowercased by `MasteryConfig.applyPackLayer`). Quests + achievements get
  their runtime id from the inner Payload's `id` field, so the asset key
  doesn't have to match.
- **CommandRewards** — one file per pack. The Payload is the full
  `{ "<SKILL_ID>": { "<level>": [rewards…] } }` map.

### Inner id casing

Quest + achievement IDs in the inner Payload (`"id": "pack_first_milestone"`)
stay **lowercase** to match how the existing plugin defaults are written
and so cross-references (`metaChildren`, `prerequisites`, `currencyId`,
`MasteryCost.currencies`) remain consistent.

## Localization (display text)

All player-facing display text ships in `Server/Languages/en-US/mmoskilltree.lang` (loaded natively via `IncludesAssetPack: true`), keyed by convention. The mod's `LocalizedText` resolver tries an explicit key field, then the by-convention key, then a raw `displayName`/`description` (the deprecated fallback) - so pack JSON should set the key and leave the literal out:

- **Mastery tracks:** `mastery.<trackId>.title` (trackId = filename lowercased). Track JSON no longer carries `displayName`. Nodes keep their template-generated `displayName` ("`<Track>: Sharpened`" etc., already DRY across the combat tracks that share `Combat6_Standard`) as the English source; translate a node by adding `mastery.<nodeId>.title`. **Node DESCRIPTIONS auto-render - do NOT author a node `description`.** The mastery page generates the effect line from the node's `modifiers` (via `MasteryModifierRenderer`, localized through `mastery.mod.*`) and renders the cost separately as chips, so a hand-written `description` (e.g. "Each purchase: +0.5%... Cost grows 10%... Requires DEFENSE level 92") is dead duplication that also bakes un-localizable balance data. Author the modifier + cost + requirements as structured fields; the text follows.
- **Quests / bounties:** `quest.<id>.title` + `quest.<id>.flavor` (id = inner `Payload.id`).
- **Currencies:** `currency.<id>.name` (id = filename lowercased) for COUNTER-backed currencies only (`mastery_point`). An ITEM-backed currency (`life_essence`) ships NO name key: with nothing authored, the mod's `CostRenderer.currencyName` derives the display name from the backing item's native, already-translated lang key (`server.items.Ingredient_Life_Essence.name`, "Essence of Life"), exactly like the icon derives from the item - zero hand-maintained translations, and the currency can never disagree with the inventory tooltip.
- **Achievements:** `achievement.<id>.title` + `achievement.<id>.desc`.
- **Token Shop offers:** explicit `"titleKey"`/`"descriptionKey"` in the offer JSON pointing at `.lang` keys.

Translate by shipping `Server/Languages/<bcp47>/mmoskilltree.lang` with the same keys (missing keys fall back to English per key). Reward line-items carry NO `displayName`: the mod auto-renders a localized line from the reward itself ("+5 Mastery Points", "Taming XP x2 for 45m", native item names), so a baked-in literal would double-render the amount AND pin the text to English. No em-dashes in `.lang` values.

## Mastery template extension (plugin 1.1.0+)

Pack-shipped mastery tracks can extend a reusable skeleton instead of
authoring every node from scratch. Templates live in
`Server/MMOSkillTree/MasteryTemplates/*.json` and load before tracks via the
`MasteryTemplateAsset` store. The canonical template `Combat6_Standard.json`
captures the shared 6-node combat shape (T1×2 choice, T2×2 choice, T3
capstone with level-70 gate, T9 eternal with level-92 gate + 1 mastery_point
floor cost). 11 of the 14 skill-scoped combat tracks use it; the 9 non-combat
shapes (gathering trio + ability tracks) stay hand-authored.

Track payload using a template:

```json
{
  "Name": "Archery_Mastery",
  "Payload": {
    "extends": "combat6_standard",
    "target": "skill:ARCHERY",
    "displayName": "Archery Mastery",
    "params": {
      "prefix": "arc",
      "gateSkill": "ARCHERY",
      "trackName": "Archery Mastery",
      "trackId": "archery_mastery",
      "dmgIngredient": "Ingredient_Lightning_Essence",
      "comboIngredient": "Ingredient_Lightning_Essence",
      "combatTarget": "ARCHERY",
      "sacrificeStat": "STAMINA",
      "capDisplay": "Eagle Eye",
      "skillLower": "archery"
    },
    "nodeOverrides": {
      "arc_t1_dmg": {
        "cost": { "items": [{ "id": "Ingredient_Lightning_Essence", "count": 6 }] }
      }
    },
    "extraNodes": [
      { "id": "arc_unique_capstone", "tier": 10, "displayName": "...", "cost": {...}, "modifiers": [...] }
    ]
  }
}
```

Resolution semantics (see `MasteryTemplateResolver.java`):

1. **Deep-clone** the template Payload.
2. **`{{paramName}}` substitution** — walks every string value recursively;
   replaces `{{key}}` with `params.get(key)`. **Empty param drops the
   holding key entirely** — lets a track opt out of optional template fields
   (e.g. Magic/Artillery pass `combatTarget: ""` so the resolved modifiers
   have no `combatTarget` key).
3. **Track-level fields overlay the template** — everything in the track
   Payload except `extends`/`params`/`nodeOverrides`/`extraNodes` wins
   (`target`, `displayName`, `icon`, `refundPercent`, track-level
   `requirements`).
4. **`nodeOverrides`** — for each `nodeId → overrideObject`, find the node
   in the resolved `nodes` array (match by post-substitution `id`) and
   **deep-merge**: object keys merge recursively; primitives + arrays
   replace wholesale. Used for per-track asymmetries (Archery's T1 dmg
   ingredient count = 6 vs the template default of 1).
5. **`extraNodes`** — append wholly new node objects after the template's
   nodes. Track-supplied id must NOT collide with a template id (use
   `nodeOverrides` to modify existing nodes). Used for track-unique
   content (Magic's `mag_archmagus` L100 capstone).

A missing param surfaces as a literal `{{key}}` in the resolved JSON
(never silently dropped) so content typos are visible during validation.
Unknown templates log a warning and the track is dropped from the effective
set. Template id lookup is case-insensitive.

The 11 thin combat tracks are typically ~20–40 lines each (vs ~210 lines
pre-template). The pack `Control/MMOSkillMasteryPack.json` adds
`"MasteryTemplates": "add"` alongside the existing content-type modes.

## CommandReward template extension (plugin 1.1.0+)

Pack-shipped CommandRewards can extend reusable per-skill-block templates.
Templates live in `Server/MMOSkillTree/CommandRewardTemplates/*.json` and
load before CommandRewards. A template Payload is a level&rarr;rewards map
(same shape a single skill block carries) optionally with `{{paramName}}`
substitution tokens:

```json
{
  "Name": "MasteryPointMilestones",
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

1. **`extends: "<template-id>"`** — pull the template Payload (case-insensitive id lookup).
2. **`params: { ... }`** — feed `{{paramName}}` substitution. Empty resolve drops the holding key.
3. **`levelOverrides: { "<level>": [rewards] }`** — replace a level's reward
   array wholesale. The level must exist in the template — unknown levels
   warn-and-skip (use `extraLevels` for new ones).
4. **`extraLevels: { "<level>": [rewards] }`** — add new levels. Collision
   with a template level warns-and-skips (use `levelOverrides` to modify).
5. Any non-reserved top-level field overlays the template.

The sentinel only fans out to skill keys — `TOTAL` and `GLOBAL_SKILL`
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
   description, rewards array as a whole, etc. — so the easiest way to
   replace rewards is to supply a fresh `rewards: [...]`).
4. `*Overrides` deep-merge into the matching array entry by `id`.
5. `extra*` append new entries (collision with template id is rejected).

The pack `Control/MMOSkillMasteryPack.json` adds `"QuestTemplates": "add"`
and `"AchievementTemplates": "add"` alongside the existing content-type modes.

The mastery pack ships an example template directory ready to populate
(empty by default — the 5 quests + 5 achievements that ship today are
hand-authored; templates pay off once a quest/achievement family grows
beyond ~3 near-identical instances). `Pack_Essence_Hoarder_100.json` /
`Pack_Essence_Hoarder_1000.json` are the natural first template candidates
(identical shape, only `requiredAmount` / `points` / `displayName` differ).

## Editing the pack content

- **Mastery templates** (`Server/MMOSkillTree/MasteryTemplates/*.json`) —
  see "Mastery template extension" above. Edit the template to change the
  shared shape across all tracks that extend it. Track-level overrides
  (`nodeOverrides`/`extraNodes`) win; per-track param values plug into
  `{{tokens}}`.
- **Mastery tracks** (`Server/MMOSkillTree/Masteries/*.json`) — schema
  documented by example in `mods/mmoskilltree/_reference/defaults-mastery.json`
  on a running server. Track id = filename (lowercased); inner Payload has
  `target` (skill:X or ability:X), `displayName`, optional `requirements`,
  `nodes` array. Each node has `id`, `tier`, `displayName`, `cost` (currency
  map + optional items + optional statSacrifice), `modifiers` (AbilityModifier
  array). Repeatable Eternal nodes require `maxPurchases: -1` (or >1) +
  `costScaling`. **Tracks may also use `extends`** to pull from a template
  — see "Mastery template extension".
- **Currencies** (`Server/MMOSkillTree/Currencies/*.json`) — id = filename
  lowercased. Two flavors: counter-backed (`hytaleItemId: null` +
  `iconItemId` for display) or item-backed (`hytaleItemId` set to a Hytale
  item like `Ingredient_Life_Essence`).
- **Quests + Achievements** — same schemas as the plugin's owner files
  under `mods/mmoskilltree/quests/` and `mods/mmoskilltree/achievements/`.
  Reward shapes include `CURRENCY` (with `currencyId` + `amount`), `XP`
  (with `skill` + `amount`), `COMMAND` (with `command` string), `BOOST_TOKEN`.

### Currency-grant command format

Always emit `/mmocurrency …` commands in the named-arg form the Hytale
parser expects. The canonical wire format is built by
`MmoCurrencyCommand.buildGiveCommand(player, currencyId, amount)` —
single source of truth for any template that needs to grant currency.

Correct:

    /mmocurrency give --player={player} --currency=mastery_point --amount=1

Wrong (legacy positional — the live command rejects it with
"Expected: 1, actual: 4" and silently grants nothing):

    /mmocurrency give {player} mastery_point 1

## Sync with plugin

The pack and the plugin co-evolve:

- If you change a mastery node's modifier shape or add a new `RewardType`,
  bump both the plugin's `MasteryConfig.SCHEMA_VERSION` (if structural) and
  re-emit the affected mastery JSON here.
- If the plugin adds a new content type (e.g. SkillTree pack support), add
  a new asset class extending `AbstractRawJsonAsset`, register in
  `AssetStoreRegistrar`, add to `PackControlAsset`, and you can immediately
  ship content of that type from this folder.

## Verification

1. Build the plugin: `./gradlew build` from the monorepo root, two levels up (`../../`). Produces
   `build/libs/MMOSkillTree-X.Y.Z.jar`.
2. Build the pack zip (see Build & deploy above).
3. Copy both into your Hytale mods folder
   (`D:\Games\Hytale\UserData\Mods\`).
4. Start the server. Confirm in the server log
   (`Saves/<world>/logs/<date>_server.log`):
   - `[AssetPacks] Mastery pack layer applied (23 entries, mode=add) — 20 masteries effective (3 disabled)`
   - Same for Currency, CommandRewards, Quest, Achievement layers.
   - No `Failed to decode asset:` or `Asset validation FAILED` lines.
5. In-game: open the Mastery menu tab (should be visible only if the pack
   loaded), buy a node, confirm currency deducted. Level a skill past 15,
   confirm the mastery-point reward shows as claimable.
