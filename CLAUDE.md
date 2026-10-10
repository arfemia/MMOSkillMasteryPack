# Skill Mastery Pack

Mastery tracks, the Mastery Trainer and the mastery currency. The family-wide rules apply here; this file adds only what is specific to this pack.

- A track's `Parent` names the base's exact filename (`"Mastery_Base"`); a generator's `Base` names the lower-cased id (`"mastery_base"`). A miss reports `UNKNOWN_BASE`.
- Purchases are saved as `<trackId>:<nodeId>`, so renaming a track file or a node id orphans every buyer.
- Never author a node `DescriptionKey`: the mastery page renders the effect line from `Modifiers` and the cost as chips.
- A per-ability modifier `Key` works only if that ability's `Body` declares it (fireball has `damageRadius`, not `radius`); an undeclared key is silently inert.
- `Life_Essence.json` ships identically here and in the bounty pack; keep the two files identical.
- `Shop_Convert_Mastery_Point.json` keeps its odd id because players' purchase counts are filed under it. It lists in the bounty pack's General storefront, so without that pack it exists with no shelf.
- This pack owns the Mastery Trainer end to end (the jar ships none); add every quest the trainer gives to the trainer dialogue's `Start.Quests`.
- The mastery point ladder is `Achievements/MMOSkillTree/Mastery/PerSkill/`: `Mastery_Point_Base` plus one short child per rung through `Parent`. `Abstract` never inherits, so every rung sets it with its own `PerSkill.IdPattern`; the stamped ids `{skill}_mastery_<N>` are saved progress, so never rename one.
- Acrobatics and Fishing ship parked; Enchanting ships on (Void essence, one Voidheart at the third step). The Utility generator switches each row through a per-row `{enabled}` token, not a whole-generator `Enabled: false`.
- A generator value that is exactly one token keeps its bound JSON type (`"Enabled": "{enabled}"` lands as a boolean, `"Count": "{t3ExtraCount}"` as a number).
