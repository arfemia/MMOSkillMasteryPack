# Changelog - MMO Skill Mastery Pack

## [2.0.0] - UNRELEASED (HELD)

- **Every mastery file rewritten onto the plugin's structured mastery schema.** Two `Abstract` bases (`Mastery_Base`, `Mastery_School_Base`) reused through native `Parent`, fifteen hand-authored tracks, and the three per-skill families (combat, gathering, utility) each written once as a `MasteryGenerators/` file the plugin stamps out per skill. The old template files are gone.
- **Track content is unchanged in play.** Every node id, price, gate and bonus carries across, so purchased nodes, respec state and stat sacrifices are untouched.
- **Modifier entries author the canonical PascalCase grammar**: `ParamKey` (the older `Key` spelling still parses), the `SET` shape where a value is replaced outright, and a `Condition` carried directly on the modifier it gates (nested `SelfHp`/`TargetHp` groups) instead of a wrapper entry.
- **The Fireball pierce node (Burning Pierce) grants +1 pierce on top of Fireball's own base**, stacking with any other pierce bonus, instead of overwriting the ability's pierce total.
- **The pack's quests and achievements move onto the shared progression shape**: an achievement's `Criteria` is a map keyed by criterion id, and every payout goes through the one `Rewards` group (`Auto` pays on the spot, `Claim` waits to be collected). Behavior-preserving; nobody's progress is re-pointed.
- Requires MMO Skill Tree 1.6.0+ and ZiggfreedCommon 2.0.0+. A pre-2.0.0 copy of this pack has no readable tracks on those plugin versions; update the pack and the plugin together.
