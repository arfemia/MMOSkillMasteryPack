# MMO Skill Mastery Pack

**The mastery and currency content pack for the [MMO Skill Tree plugin](https://www.curseforge.com/hytale/mods/mmo-skill-tree).** Drop this `.zip` into your server's `mods/` folder alongside the plugin to unlock the full mastery system - 20 active tracks across combat, ability, and gathering skills, two starter currencies, mastery-point milestone rewards every 15 levels, and themed quests + achievements that exercise the mastery surface.

---

## What's inside

### Mastery tracks (20 active)

- **11 combat-skill tracks** - `swords_mastery`, `daggers_mastery`, `polearms_mastery`, `staves_mastery`, `axes_mastery`, `blunt_mastery`, `unarmed_mastery`, `archery_mastery`, `magic_mastery`, `artillery_mastery`, `defense_mastery`. Each opts into swing damage via `combatTarget`.
- **6 marquee ability tracks** - `fireball_mastery`, `meteor_mastery`, `whirlwind_mastery`, `piercing_shot_mastery`, `shield_slam_mastery`, `shadowstep_mastery`. Narrower buffs, higher per-purchase Eternal rates than skill-scope tracks.
- **3 gathering tracks** - `mining_mastery`, `woodcutting_mastery`, `harvesting_mastery`. Eternals use the `lootMultiplier` paramKey so they stay valuable past L100.

Three additional tracks (`fishing_mastery`, `enchanting_mastery`, `acrobatics_mastery`) ship in the zip with `"disabled": true` - parked for balance work and will activate in a future pack release without a plugin update.

Each track has **3-5 finite identity nodes** that define its character, plus exactly one infinitely-repeatable **"Eternal" node** (tier 9) for multi-year progression. Identity cost curve: combat + marquee ability tracks use T1 = 600 Life Essence, T2 = 1500, T3 = 4000 (Meteor uniquely has a T4 at 8000); gathering tracks are lighter at T1 = 500, T2 = 1200, T3 = 3500. Eternals start cheap (200-500 LE depending on track) and scale 1.10× per purchase (Fireball uses 1.25× and Meteor soft-caps at 70 buys) - self-limiting curves where each player naturally plateaus.

### Currencies (2)

- **Mastery Point** (`mastery_point`) - counter-backed (a pure number on the player, can't be dropped, traded, or lost). The pacing currency for finite identity progression - earned every 15 levels from any skill, claimable from that skill's rewards.
- **Life Essence** (`life_essence`) - item-backed (wraps Hytale's native `Ingredient_Life_Essence`). Stackable, tradeable, drops on death. The bulk currency for Eternal node grinds.

### Mastery-point command rewards

Adds a manual-claim Mastery Point reward to every 15th level for all 20 built-in skills, generated up to each skill's configured max level. Rewards self-hide when the Mastery Points currency is disabled.

The pack uses the new **CommandReward template system** (plugin 1.1.0+): one reusable `MasteryPointMilestones` template (12 milestone levels) is referenced from a single `{{ALL_SKILLS}}` entry that fans out to every skill. Going from 240 identical reward objects to 1 template + 1 reference. Server owners who want different intervals or amounts can override per skill - explicit skill entries win over the catch-all.

### Themed quests (5)

- **First Step Toward Mastery** - auto-accept on first level-up. Reach level 15 in any skill → 2 Mastery Points.
- **Essence Collector** - turn in 100 Life Essence → 5 Mastery Points + 2,000 Mining XP + 2,000 Woodcutting XP.
- **Essence Devotee** (sequential, prereq Essence Collector) - turn in 500 Life Essence → 25 Mastery Points.
- **Path of Mastery** - reach L25 in Mining, Woodcutting, Harvesting, Swords, Archery → 10 Mastery Points.
- **Mastery Tithe** (auto-accept, repeatable 6h, endgame) - slay 10 mobs + chop 10 logs → 1 Mastery Point. Gated behind Total Level 500 and the **Mastery Hoarder** achievement, so the trickle stays an endgame faucet rather than an early-game boost.

### Themed achievements (6)

- **Apprentice of the Path** (25 pts, +1 MP) - reach L15 in any skill.
- **Journeyman of the Path** (100 pts, +3 MP) - reach L60 in any skill.
- **Essence Hoarder I** (25 pts, +2 MP) - pick up 100 Life Essence.
- **Essence Hoarder II** (150 pts, +10 MP) - pick up 10,000 Life Essence.
- **Mastery Hoarder** (50 pts, +5 MP) - earn 300 Mastery Points lifetime. Counts cumulatively across milestone rewards, quests, and admin grants; refunds don't subtract.
- **Devotee of Mastery** (250 pts meta, +20 MP) - unlock the four non-meta achievements above.

### Shop offer (plugin 1.3.0+)

Running the [MMO Skill Bounty Pack](https://www.curseforge.com/hytale/mods/mmo-skill-tree-bounty-board-pack-rich-customizable) too? This pack adds a **Mastery Infusion** offer to the bounty pack's shop: convert Bounty Tokens into a Mastery Point (capped per day). It appears only when both packs are installed on plugin 1.3.0+ (the shop engine); without the Bounty Pack it stays dormant.

### Fully translated (9 languages)

Every display string in the pack - track titles, node names and descriptions, quest titles and flavor, achievement names, currency names, and the shop offer - ships in English, German, Spanish, French, Hungarian, Italian, Brazilian Portuguese, Russian, and Turkish. Node tier names ("Sharpened", "Vampiric", "Eternal", ...) are translated once per tier and the plugin (1.3.0+) composes them with the localized track title, so they stay translated even on custom-skill tracks you fan out yourself. Any key a translation misses falls back to English per key.

## Installation

1. Install the [MMO Skill Tree plugin](https://www.curseforge.com/hytale/mods/mmo-skill-tree).
2. Drop `MMOSkillMasteryPack-1.1.0.zip` into the same `mods/` folder.
3. Restart the server.

On startup, look for:

```
[AssetPacks] Mastery pack layer applied (20 entries, mode=add) - 20 masteries effective
[AssetPacks] Currency pack layer applied (2 entries, mode=add)
[AssetPacks] CommandRewards pack layer applied (1 packs, mode=add) - N skill+level entries effective
[AssetPacks] Quest pack layer applied (5 entries, mode=add)
[AssetPacks] Achievement pack layer applied (6 entries, mode=add)
```

## Without this pack

The MMO Skill Tree plugin still runs - XP, skill tree, skill rewards, quests, and achievements all work normally. The Mastery and Currency menu tabs hide themselves on an empty track/currency set (`MasteryConfig.isAvailable()` / `CurrencyConfig.isAvailable()` gate on emptiness), so the absence is graceful, not broken.

## For server owners - customizing

- The mastery tracks, currencies, quests, and achievements ship as the pack's `add`-mode contribution for those types, but the **server owner's own files in `mods/mmoskilltree/`** always win. Edit `mods/mmoskilltree/mastery.json` to retune nodes; the pack's defaults fall through where you don't override.
- The mastery-point milestone rewards are in this pack's `Server/MMOSkillTree/CommandRewards/MMOSkillMasteryPack.json`. To retune intervals or amounts, override entries in your own `command-rewards.json` - your overrides win.

## For pack authors - building your own

The pack uses the standard MMO content-pack format documented in the plugin's `CONTENT_PACKS.md` (alongside the plugin source). To extend or replace what this pack ships, create your own pack with the matching content types - by default new packs **add** alongside this one; use `Control/<yourpack>.json` to declare `replace` mode per type for total-conversion packs.

## Versions

| Pack  | Plugin | Notes                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| ----- | ------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.1.0 | 1.3.0+ | Full 9-language localization of every display string (track titles, node names + descriptions including the shared tier names, quests, achievements, currency names, shop offer). Fixes the Mastery Infusion offer never actually appearing in the shop (1.0.1 shipped it in the wrong content folder with a stale Control key). Reward lines drop their baked English names; the plugin renders localized amounts instead. Adds a **Mastery Infusion** shop offer (convert Bounty Tokens into a Mastery Point), active when the Bounty Pack and plugin 1.3.0+ are present. Core mastery content unchanged. |
| 1.0.0 | 1.1.0+ | First standalone release. Migrates `MasteryDefaults` + `CurrencyDefaults` + mastery-point milestones out of the plugin jar. Mastery-point milestone file uses the plugin's CommandReward template + `{{ALL_SKILLS}}` system (plugin 1.1.0+) - 1 template + 1 sentinel entry covers all 20 skills.                                                                                                                                                                                                                                                                                                           |

## Roadmap

- More themed quests + achievements tied to specific mastery tracks.
- Higher-tier mastery nodes targeting endgame combat.
- More currencies for different progression axes.

---

## [Plugin homepage](https://www.curseforge.com/hytale/mods/mmo-skill-tree) - [Plugin docs](https://mmo-skill-tree-docs.ziggfreed.com) - [Discord](https://discord.gg/5NFdZsUxHZ)
