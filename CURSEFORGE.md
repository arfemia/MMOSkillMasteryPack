# MMO Skill Mastery Pack

**The mastery and currency content pack for the [MMO Skill Tree plugin](https://www.curseforge.com/hytale/mods/mmo-skill-tree).** Drop this `.zip` into your server's `mods/` folder alongside the plugin and the full mastery system switches on: 28 active tracks across combat skills, marquee abilities, gathering, Enchanting and seven of the damage schools, plus two starter currencies, a claimable Mastery Point every 15 levels, the Mastery Trainer's daily lessons, and themed quests and achievements to earn them with.

[![Discord](https://img.shields.io/badge/Discord-Join%20Server-5865F2?style=for-the-badge&logo=discord&logoColor=white)](https://discord.gg/5NFdZsUxHZ) [![Ko-fi](https://img.shields.io/badge/Ko--fi-Support-FF5E5B?style=for-the-badge&logo=ko-fi&logoColor=white)](https://ko-fi.com/ziggfreed) [![Documentation](https://img.shields.io/badge/Docs-Read%20More-0ea5e9?style=for-the-badge)](https://mmo-skill-tree-docs.ziggfreed.com)

---

[![Host your own Hytale server with Kinetic Hosting](https://i.imgur.com/UHn3FzW.png)](https://billing.kinetichosting.com/aff.php?aff=1262)

---

## What's inside

### Mastery tracks (28 active)

- **11 combat-skill tracks** - Swords, Daggers, Polearms, Staves, Axes, Blunt, Unarmed, Archery, Magic, Artillery, and Block. Each buffs the skill's ordinary swings as well as its abilities.
- **6 marquee ability tracks** - Fireball, Meteor, Whirlwind, Piercing Shot, Shield Slam, and Shadowstep. Narrower buffs, with Eternals that grow faster per purchase.
- **3 gathering tracks** - Mining, Woodcutting, and Harvesting. Their Eternals boost gathering loot, so they stay valuable past level 100.
- The Enchanting track: Enchanting loot luck, priced in Essence of the Void. It opens at Enchanting level 5.
- 7 damage-school tracks - Fire, Ice, Lightning, Water, Arcane, Void, Poison. A school node follows the damage, not the weapon: buy Fire and it pays on a fireball, on a meteor, and on a molten hammer swing. Each school also sells resistance to itself, so you can armour up against the element that keeps killing you.

School tracks run ten nodes. Two openers (percent damage or flat damage), a resistance step, three flavor nodes tuned to that school's own spells, a capstone that costs a permanent slice of your max Health, Stamina, or Mana, an Ascendant pair (school-wide cooldown cuts, or another slab of flat damage), and the usual Eternal at +1% a purchase. Every gate reads combat level (16 to 92) instead of one skill, so a mage and a rogue climb the same ladder.

Two more tracks (Fishing, Acrobatics) ship in the zip switched off, parked for balance work until a later release; a server owner can turn one on early from `mastery.json`.

Each track has **3-5 finite identity nodes** that define its character, plus exactly one infinitely-repeatable **"Eternal" node** (tier 9) for multi-year progression. Identity cost curve: combat + marquee ability tracks use T1 = 900 Essence of Life, T2 = 2250, T3 = 6000 (Meteor uniquely has a T4 at 12000); gathering tracks are lighter at T1 = 750, T2 = 1800, T3 = 5250. Eternals start cheap (300-750 Essence of Life depending on track) and scale 1.10x per purchase (Fireball uses 1.25x and Meteor soft-caps at 70 buys) - self-limiting curves where each player naturally plateaus.

### Currencies (2)

- **Mastery Point** (`mastery_point`) - counter-backed (a pure number on the player, can't be dropped, traded, or lost). The pacing currency for finite identity progression - earned every 15 levels of any skill and collected from your achievement book.
- **Essence of Life** (`life_essence`) - item-backed (wraps Hytale's native `Ingredient_Life_Essence`). Stackable, tradeable, drops on death. The bulk currency for Eternal node grinds.

### A Mastery Point every 15 levels

Every skill pays a Mastery Point at level 15, 30, 45 and on up to 195. Each one waits in the Mastery section of your achievement book until you collect it, with each skill's levels listed together. Skills a pack adds get the same ladder.

Already past some of those levels when the pack goes in? Their points are waiting for you the next time you log in. If the server turns mastery off, the whole ladder hides.

### The Mastery Trainer's lessons

Once you've met the Mastery Trainer in the Forgotten Temple (or collected What You Already Paid), ask "Do you have lessons for me?" and the lessons board opens. New lessons go up every day, and all of them are done out in the world: fell logs, mine ore, catch fish, put down creatures, use your abilities, craft at a bench. Bring each one back to the board to collect a handful of Essence of Life and standing with the trainer.

Your standing with the trainer, Mastery Trainer's Favor, climbs from Newcomer to Grandmaster, and each rank from Student up pays Mastery Points:

- Student (1,000 standing): 3 points, and a Crafting lesson opens on the board.
- Trusted Student (3,000): 5 points.
- Adept (9,000): 10 points and a Hard lesson.
- Master (21,000): 15 points and a Master's lesson, which pays a Mastery Point every day you do it.
- Master II (30,000) pays 15 points and Master III (40,000) pays 20.
- Grandmaster (50,000): 25 points, the Grandmaster title and a second Master's lesson.

A rank's points wait in the Mastery section of your achievement book until you collect them. Ask "How am I doing?" to see your rank and what the next one opens. Doing every lesson and The Trainer's Due each day, you reach Master in a little over two weeks and Grandmaster in about a month.

Paid the old Mastery Tithe? It's retired, and What You Already Paid shows up in your log: see the trainer and you start at Trusted Student.

### Themed quests (8)

- **A Word with the Trainer**: once you reach the Forgotten Temple, the Adventurer's Guide sends you across the landing to meet the Mastery Trainer, for 1 Mastery Point.
- **The First Milestone** (from the trainer): reach level 15 in any skill for 2 Mastery Points.
- **Essence Collector**: turn in 150 Essence of Life for 5 Mastery Points plus 2,000 Mining and 2,000 Woodcutting XP.
- **Essence Devotee** (follows Essence Collector): turn in 750 Essence of Life for 25 Mastery Points.
- **Many Trades**: reach level 25 in Mining, Woodcutting, Harvesting, Swords, and Archery for 10 Mastery Points.
- **The Trainer's Due** (daily, after A Word with the Trainer): hand in 115 Essence of Life and 3 Linen Scraps and kill 10 creatures, for an essence roll that grows with your combat level and luck, a Mastery Point about two days in five, now and then an all-skill XP boost, and rarely the Adept's Staff.
- **A Week of Lessons** (weekly, after A Word with the Trainer or What You Already Paid): finish any 5 of the trainer's lessons in a week for 1,000 standing.
- **What You Already Paid** (only if you paid the old Mastery Tithe): see the trainer for 3,000 standing.

These quests raise your standing with the trainer too: 250 for A Word with the Trainer, The First Milestone and each day's Trainer's Due, 500 for Essence Collector, 1,000 for Many Trades and 1,500 for Essence Devotee.

### Themed achievements (6)

- **Mastery Apprentice** (25 pts, +1 MP) - reach L15 in any skill.
- **Mastery Journeyman** (100 pts, +3 MP) - reach L60 in any skill.
- **Essence Hoarder I** (25 pts, +2 MP) - pick up 100 Essence of Life.
- **Essence Hoarder II** (150 pts, +10 MP) - pick up 10,000 Essence of Life.
- **Mastery Hoarder** (50 pts, +5 MP) - earn 300 Mastery Points lifetime. Counts cumulatively across milestone rewards, quests, and admin grants; refunds don't subtract.
- **Devotee of Mastery** (250 pts meta, +20 MP) - earn every other mastery achievement.

### Shop offer

Running the [MMO Skill Bounty Pack](https://www.curseforge.com/hytale/mods/mmo-skill-tree-bounty-board-pack-rich-customizable) too? This pack adds a **Mastery Infusion** offer to the bounty pack's shop: convert Bounty Tokens into a Mastery Point (capped per day). It appears only when both packs are installed; without the Bounty Pack it stays dormant.

### Fully translated (9 languages)

Every display string in the pack - track titles, node names and descriptions, quest titles and flavor, achievement names, currency names, and the shop offer - ships in English, German, Spanish, French, Hungarian, Italian, Brazilian Portuguese, Russian, and Turkish. Node tier names ("Sharpened", "Vampiric", "Eternal", ...) are translated once per tier and the plugin composes them with the localized track title, so they stay translated even on custom-skill tracks you fan out yourself. Any key a translation misses falls back to English per key.

## Installation

1. On a Hytale Update 7 (0.7.x) server, install the [MMO Skill Tree plugin](https://www.curseforge.com/hytale/mods/mmo-skill-tree) 1.7.1 or newer and Ziggfreed's CommonLib 2.3.0 or newer.
2. Drop `MMOSkillMasteryPack-2.2.0.zip` into the same `mods/` folder.
3. Restart the server.

On startup, look for:

```
[AssetPacks] Mastery asset layer applied (17 entries) - 15 masteries effective
[AssetPacks] Mastery generator layer applied (3 generators) - 28 masteries effective
[AssetPacks] Shared-schema achievement layer applied (N achievements) - N achievements effective
```

## Without this pack

The MMO Skill Tree plugin still runs - XP, skill tree, skill rewards, quests, and achievements all work normally. With no tracks or currencies loaded, the Mastery and Currency menu tabs hide themselves, so nothing looks broken.

## For server owners - customizing

- The mastery tracks, currencies, quests, and achievements ship as the pack's `add`-mode contribution for those types, but the **server owner's own files in `mods/mmoskilltree/`** always win. Edit `mods/mmoskilltree/mastery.json` to retune nodes; the pack's defaults fall through where you don't override.
- The Mastery Point ladder is the pack's `Mastery_Point_*` achievement files. To change what a level pays, ship a file of the same name in your own pack.
- The trainer's lessons and ranks work the same way. Each rank's pay is an achievement file and each lesson a board contract: ship a file of the same name to retune one, or a new contract that names the trainer's board to add a lesson.

## For pack authors - building your own

The pack uses the standard MMO content-pack format documented in the plugin's `CONTENT_PACKS.md` (alongside the plugin source). To extend or replace what this pack ships, create your own pack with the matching content types - new packs **add** alongside this one, a same-id mastery file in a later-loaded pack replaces this pack's track, and your tracks can name this pack's `mastery_base` / `mastery_school_base` as their own `Parent` or fan out a whole family with one `MasteryGenerators/` file.

## Roadmap

- More themed quests + achievements tied to specific mastery tracks.
- Higher-tier mastery nodes targeting endgame combat.
- More currencies for different progression axes.

---

## Links & Support

[![MMO Skill Tree](https://img.shields.io/badge/CurseForge-MMO%20Skill%20Tree-F16436?style=for-the-badge&logo=curseforge&logoColor=white)](https://www.curseforge.com/hytale/mods/mmo-skill-tree) [![Get Pro Edition](https://img.shields.io/badge/Get%20Pro%20Edition-F59E0B?style=for-the-badge)](https://mmo-skill-tree-docs.ziggfreed.com/commercial)

**Full docs:** [mmo-skill-tree-docs.ziggfreed.com](https://mmo-skill-tree-docs.ziggfreed.com) | Questions? Join the [Discord](https://discord.gg/5NFdZsUxHZ)

**Patch notes:** [every version on GitHub](https://github.com/arfemia/MMOSkillMasteryPack/tree/main/patch-notes)

**Support Development:** [Ko-fi](https://ko-fi.com/ziggfreed)

_MMO Skill Tree is not affiliated with Hypixel Studios or Hytale._
