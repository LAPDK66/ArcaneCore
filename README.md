# ✦ ArcaneCore — Paper 1.21 Plugin Design Document

> **Target:** Paper MC 1.21
> **Author:** LAPDK66

---

## Table of Contents

1. [Plugin Overview](#1-plugin-overview)
2. [Magic Types](#2-magic-types)
3. [Mana System](#3-mana-system)
4. [Level & Skill System](#4-level--skill-system)
5. [Rune System](#5-rune-system)
6. [Spell System](#6-spell-system)
7. [Crafting Blocks & Recipes](#7-crafting-blocks--recipes)
8. [Ritual Crafting System](#8-ritual-crafting-system)
9. [Magic Robes (Armor)](#9-magic-robes-armor)
10. [Artifact System](#10-artifact-system)
11. [Stardust System](#11-stardust-system)
12. [World Structures](#12-world-structures)
13. [UI & Menus Summary](#13-ui--menus-summary)
14. [God Armor](#14--god-tier-magic-armor-magic-star-enhancement)
15. [Balance Notes](#15-balance-notes)

---

## 1. Plugin Overview

**ArcaneCore** is a comprehensive magic-themed expansion plugin for Paper 1.21. It introduces a full skill & progression system, new crafting paradigms through ritual magic, a rune-based spell crafting system, special armor with passive magic effects, equippable artifacts with their own inventory menu, a stardust enhancement system for armor and tools, and new world structures scattered across the overworld and beyond.

**Core Pillars:**

- **Discover** — Find runes, artifacts, and structures in the world.
- **Progress** — Level up, spend skill points, unlock stronger spells.
- **Craft** — Use ritual tables and runic combinations to forge spells and equipment.
- **Enhance** — Convert old items into Stardust and infuse your gear with it.

---

## 2. Magic Types

All spells, robes, and skill upgrades belong to one of the following **Magic Types**. Each type has its own level (see Section 4), its own robe set (see Section 9), and its own rune variant (see Section 5).

| ID | Magic Type | Category | Theme / Flavor |
|----|------------|----------|----------------|
| 1 | **Fire** | Standard | Destruction, heat, explosions |
| 2 | **Water** | Standard | Flow, frost, healing |
| 3 | **Earth** | Standard | Defense, nature, gravity |
| 4 | **Air** | Standard | Speed, lightning, wind |
| 5 | **Poison** | Standard | DoT, debuffs, withering |
| 6 | **Holy / Light** | ⚠ Restricted | Healing, shields, radiance, undead damage |
| 7 | **Dark** | ⚠ Restricted | Corruption, curses, shadow strikes, vampirism |

> **Note to Developers:** The magic type list can be expanded later. All systems reference types by their ID, so adding new ones is non-breaking.

### 2.1 Standard vs. Restricted Magic Types

Types 1–5 (Fire, Water, Earth, Air, Poison) are **Standard** types. Any player can freely invest Skill Points in them and use spells of any of these types simultaneously.

Types 6 and 7 (**Holy/Light** and **Dark**) are **Restricted** types with the following special rules:

#### Light & Dark — Special Rules

**1. Mutual Exclusivity (Default)**
By default, a player may only have **one** of Light or Dark active at a time. The active alignment is called the player's **Affinity**. Affinity is chosen the first time a player spends a Skill Point in either Light or Dark.

- Skill Points invested in the inactive Affinity are **locked** and do not provide any effect until the player switches.
- Equipped spells of the inactive Affinity cannot be cast (they show a red error on the action bar: `✦ Your current Affinity does not allow this.`).

**2. Switching Affinity**
A player can switch their active Affinity by using the **Affinity Switch** option in the Skill Menu. After switching:
- A **30-minute real-time cooldown** is applied before the player can switch again.
- The cooldown is stored persistently (survives server restarts and relog).
- During the cooldown, the action bar shows: `✦ Affinity Locked — Switch available in: 27:43`

**3. Higher Requirements**
Light and Dark spells are inherently more powerful and therefore more demanding:

| Restriction | Light | Dark |
|-------------|-------|------|
| Base Mana Cost Multiplier | ×1.5 | ×1.5 |
| Minimum Magic Power Level per Tier | +2 above normal | +2 above normal |
| Starting Skill Point to unlock | 5 SP (one-time gate) | 5 SP (one-time gate) |

For example, a Tier III spell normally requires Magic Power Level 5. A Tier III **Light** or **Dark** spell requires Magic Power Level **7**.

**4. Affinity Switch Upgrade (Tiered Unlock)**
Rather than a single expensive unlock, the Affinity switch system is a **3-rank upgrade** in the General skill tab, with each rank reducing the cooldown further until both types can be used simultaneously:

| Rank | Cooldown | Total SP Cost |
|------|----------|---------------|
| 0 (default) | 30 minutes | — |
| 1 | 15 minutes | 10 SP |
| 2 | 5 minutes | 30 SP |
| 3 | No cooldown — simultaneous use | 60 SP |

Each rank costs more than the last (10 → 20 → 30 SP), and Rank 3 requires at least 5 SP already spent in both Light and Dark. See Section 4.4 for full details.

---

## 3. Mana System

### 3.1 What is Mana?

Mana is the fuel for casting spells. Every player has a **Mana Pool** displayed on the action bar (or a custom HUD element). Mana regenerates over time passively.

### 3.2 Base Stats

| Stat | Default Value |
|------|---------------|
| Base Max Mana | 100 |
| Mana Regen per second | 2 |
| Mana Regen delay after cast | 3 seconds |

### 3.3 Mana Modifiers

Mana can be increased through:

- **Skill Points** spent in the "Mana Pool" upgrade branch (see Section 4).
- **Magic Robe pieces** worn — each worn piece grants a flat mana bonus (see Section 9).
- **Artifacts** with mana-related affixes (see Section 10).

### 3.4 Mana Display

Mana is shown as a color-coded action bar message:
`✦ Mana: 80 / 150 ✦`

The color shifts from blue (full) → yellow (mid) → red (low).

---

## 4. Level & Skill System

### 4.1 Player Level

Players earn **XP** through casting spells, discovering structures, crafting at ritual tables, and defeating mobs near shrines. On level-up, the player receives **Skill Points (SP)**.

| Level Bracket | SP per Level-Up |
|---------------|-----------------|
| 1 – 10 | 1 SP |
| 11 – 25 | 2 SP |
| 26 – 50 | 3 SP |
| 51+ | 4 SP |

### 4.2 Skill Menu (GUI)

Opened with `/skills` or a bound key item (e.g., right-clicking the **Skill Tome** item). The GUI is a chest-based inventory with tabs along the top row. Each tab corresponds to an upgrade category.

**Tabs:**

| Tab | Icon | Description |
|-----|------|-------------|
| General | Book | Global upgrades (mana pool, regen, cast speed, Affinity Switch) |
| Fire | Blaze Powder | Fire magic power + fire-specific upgrades |
| Water | Water Bucket | Water magic power + water-specific upgrades |
| Earth | Grass Block | Earth magic power + upgrades |
| Air | Feather | Air magic power + upgrades |
| Poison | Spider Eye | Poison magic power + upgrades |
| ☀ Light | Glowstone Dust | Light magic power + upgrades *(locked if Dark is active Affinity)* |
| ☽ Dark | Coal | Dark magic power + upgrades *(locked if Light is active Affinity)* |

> The Light and Dark tabs always show all upgrades, but nodes in the **inactive** Affinity's tab appear grayed out with a lock icon and a tooltip: `"Switch your Affinity to invest here."`

### 4.3 Scaling Cost System

Every upgrade node in the entire Skill Menu uses **scaling costs per rank**. Each rank costs more than the last according to this formula:

> **Cost of Rank N = Base Cost × N**

So a node with Base Cost 2 SP costs: 2 → 4 → 6 → 8 → 10 SP for ranks 1–5. The first rank is always cheap; deep investment becomes progressively more expensive. This applies to **every node** across all tabs.

**Example — [Fire] Power (Base: 2 SP, Max: 10 Ranks):**

| Rank | Cost | Cumulative SP Spent |
|------|------|---------------------|
| 1 | 2 SP | 2 SP |
| 2 | 4 SP | 6 SP |
| 3 | 6 SP | 12 SP |
| 4 | 8 SP | 20 SP |
| 5 | 10 SP | 30 SP |
| 6 | 12 SP | 42 SP |
| 7 | 14 SP | 56 SP |
| 8 | 16 SP | 72 SP |
| 9 | 18 SP | 90 SP |
| 10 | 20 SP | 110 SP |

Maxing a single type's Power tree alone costs **110 SP**. This ensures no player can max everything — choices matter.

### 4.4 General Upgrade Branch

| Upgrade Node | Base Cost | Max Rank | Effect per Rank |
|--------------|-----------|----------|-----------------|
| Expanded Mana | 1 SP | 10 | +25 Max Mana |
| Swift Recovery | 2 SP | 5 | +0.5 Mana/s regen |
| Cast Efficiency | 3 SP | 3 | -5% spell mana cost |
| Resilient Caster | 2 SP | 3 | +5% damage reduction while casting |
| **Affinity Switch** | **10 SP** | **3** | See table below |

> All costs scale per rank using the formula above. E.g. Expanded Mana costs 1 SP for rank 1, 2 SP for rank 2, up to 10 SP for rank 10 (110 SP total to max).

**Affinity Switch Upgrade — Tier Breakdown:**

| Rank | Cost (this rank) | Cumulative SP | Effect |
|------|-----------------|---------------|--------|
| 0 *(default)* | — | — | 30-minute cooldown between Affinity switches |
| 1 | 10 SP | 10 SP | Cooldown reduced to **15 minutes** |
| 2 | 20 SP | 30 SP | Cooldown reduced to **5 minutes** |
| 3 | 30 SP | 60 SP | **Simultaneous use** — both Light and Dark are fully active at all times; no cooldown; no Affinity restriction |

> **Rank 3 Prerequisite:** The player must have spent at least **5 SP in both** the Light tree AND the Dark tree before Rank 3 becomes available. Ranks 1 and 2 have no prerequisite.

> The Affinity Switch upgrade is shown at the bottom of the General tab, visually separated from the other nodes with a divider line. All three ranks are visible at once so players can plan the full investment.

### 4.5 Standard Magic Type Upgrade Branch (Fire / Water / Earth / Air / Poison)

Each of the 5 standard types has its own sub-tree. The **Power Level** of a magic type determines which spells of that type you can equip and how much bonus damage/effect scaling they receive.

| Node | Base Cost | Max Rank | Effect per Rank |
|------|-----------|----------|-----------------|
| [Type] Power | 2 SP | 10 | +1 Magic Power Level for this type |
| [Type] Mastery | 3 SP | 5 | +8% effectiveness of [Type] spells |
| [Type] Reduction | 2 SP | 3 | -10% mana cost of [Type] spells |
| [Type] Affinity | 4 SP | 1 | Passive bonus matching the magic type (e.g., Fire Resistance for Fire) |

> All costs scale per rank. [Type] Affinity has only 1 rank so its cost is simply its base cost (4 SP).

### 4.6 Restricted Magic Type Upgrade Branch (Light / Dark)

Light and Dark share the same node structure but have a **steeper base cost** at every node and a **5 SP one-time entry gate**. The Affinity Switch button in the tab (not a spendable upgrade — just a button) triggers the cooldown defined by the Affinity Switch upgrade rank from Section 4.4.

| Node | Base Cost | Max Rank | Effect per Rank |
|------|-----------|----------|-----------------|
| [Type] Unlock | **5 SP (flat, one-time)** | 1 | Unlocks the Light or Dark tree; sets this as active Affinity if first time |
| [Type] Power | **3 SP** | 10 | +1 Magic Power Level for this type |
| [Type] Mastery | **4 SP** | 5 | +8% effectiveness of [Type] spells |
| [Type] Reduction | **3 SP** | 3 | -10% mana cost of [Type] spells (stacks on top of the base ×1.5 cost) |
| [Type] Affinity Passive | **5 SP (flat, one-time)** | 1 | Passive bonus: Holy → Regeneration I permanent; Dark → 3% lifesteal on spell hit |
| *(Switch button)* | *(not a node)* | — | Triggers Affinity switch; cooldown is determined by Affinity Switch upgrade rank |

> The **Unlock** and **Affinity Passive** nodes do not scale (they are single-rank purchases at a flat cost). All multi-rank nodes use the scaling formula.

**Magic Power Level** is the gating stat for equipping spells. A spell with a **Level Requirement of 3** normally requires Power Level 3 in that type. Light and Dark spells require Power Level **+2 higher** than their tier would normally demand (see Section 2.1).

---

## 5. Rune System

### 5.1 Overview

Runes are physical items found by exploring the world (shrines, magic houses, dungeon chests, boss drops). They cannot be crafted. Runes are the core ingredients for crafting spells at the Ritual Table.

### 5.2 The Four Rune Types

Every spell requires exactly **one rune of each of the four types**. The combination determines the resulting spell.

---

#### Type A — Magic Rune (Element Rune)
Defines **which magic type** the spell belongs to.

| Rune Name | Magic Type | Category |
|-----------|------------|----------|
| Ember Rune | Fire | Standard |
| Tide Rune | Water | Standard |
| Stone Rune | Earth | Standard |
| Gale Rune | Air | Standard |
| Venom Rune | Poison | Standard |
| Dawn Rune | Holy / Light | ⚠ Restricted |
| Dusk Rune | Dark | ⚠ Restricted |

> **Dawn Rune** and **Dusk Rune** can only be used in the Ritual Table by a player who has that type as their **active Affinity** (or has unlocked Dual Affinity). Attempting to use the wrong rune displays: `"Your Affinity does not resonate with this rune."`

---

#### Type B — Effect Rune (Delivery Rune)
Defines **how** the spell is delivered or what it does at a base level.

| Rune Name | Effect |
|-----------|--------|
| Arrow Rune | Projectile (fires a magic bolt) |
| Burst Rune | AoE explosion around caster |
| Beam Rune | Continuous beam / ray |
| Nova Rune | Circular shockwave expanding outward |
| Mark Rune | Place a delayed trap on the ground |
| Surge Rune | Dash / movement in target direction with effect |
| Sigil Rune | Draw a persistent rune circle on the floor |
| Veil Rune | Apply an effect to self / aura |

---

#### Type C — Modifier Rune
Modifies **how** the effect behaves (size, duration, speed, count).

| Rune Name | Modifier |
|-----------|----------|
| Swift Rune | +50% projectile / effect speed |
| Wide Rune | +75% area of effect radius |
| Long Rune | +100% duration of effects |
| Split Rune | Effect splits into 3 on impact |
| Reach Rune | +50% range / travel distance |
| Echo Rune | Spell fires a second time after 1.5s |
| Pierce Rune | Projectile passes through entities |
| Silence Rune | Removes secondary effects; costs -30% mana |

---

#### Type D — Level Rune (Power Rune)
Defines the **power tier** of the spell. Higher tier = stronger effect but requires higher Magic Power Level.

| Rune Name | Tier | Magic Power Level Required |
|-----------|------|---------------------------|
| Novice Rune | I | 1 |
| Apprentice Rune | II | 3 |
| Adept Rune | III | 5 |
| Expert Rune | IV | 7 |
| Master Rune | V | 10 |

### 5.3 How Runes are Obtained

- **Shrines** — Shrine chest loot (weighted by shrine tier).
- **Magic Houses** — Barrel and chest loot inside structure.
- **Dungeon / Stronghold chests** — Small chance of random rune.
- **Mob drops** — Rare drop from magic-themed mobs (e.g., Evokers, Endermen, Blazes drop type-appropriate runes).
- **Stardust conversion** — Runes can be converted INTO Stardust at the Stardust Maker Table; this is one-way.

---

## 6. Spell System

### 6.1 Spells as Items

Spells are custom items in the player's **hotbar or regular inventory**. Right-clicking a spell item casts it, consuming Mana. The spell item does not "break" — it has unlimited uses as long as the caster has sufficient Mana.

### 6.2 Spell Properties

| Property | Description |
|----------|-------------|
| Magic Type | Which type this spell belongs to (determines level requirement) |
| Power Level | Tier I–V, determines strength and Magic Power Level requirement |
| Delivery | How the spell is delivered (from Effect Rune) |
| Modifier | Behavioral tweak (from Modifier Rune) |
| Mana Cost | Calculated from tier and modifier (see below) |
| Cooldown | Short per-spell cooldown (0.5s base, scales with tier) |
| Damage / Effect | Scales with Magic Power Level of that type |

### 6.3 Mana Cost Formula

```
Base Cost = 10 × Tier Level (e.g., Tier III = 30 mana)
Modifier adjustment:  Swift / Silence = -5 mana; Echo / Split = +10 mana
Final Cost = Base Cost + Modifier Adjustment
```

Reduced by **Cast Efficiency** upgrades and **Magic Type Reduction** skill nodes.

### 6.4 Spell Scaling

The actual damage and effect duration of a spell scales with the player's **Magic Power Level** in that spell's type.

```
Damage = (Base Damage × Tier Multiplier) + (Magic Power Level × 0.5)
```

---

## 7. Crafting Blocks & Recipes

### 7.1 Magic Paper

**Magic Paper** is a core crafting ingredient needed for most advanced crafting. It is crafted in the **Vanilla Crafting Table**.

**Recipe (Shapeless-style, 3×3):**

```
[ Paper ][ Paper ][ Paper ]
[ Paper ][ Any Enchanted Book ][ Paper ]
[ Paper ][ Paper ][ Paper ]
```

Result: **1× Magic Paper**

> The enchanted book is consumed. Any level, any enchantment counts.

---

### 7.2 Magic Crafting Table

The central hub for all advanced crafting. All other plugin-specific tables and gear are made here.

**Recipe (3×3 Vanilla Crafting Table):**

```
[ Air ][ Magic Paper ][ Air ]
[ Diamond ][ Obsidian ][ Diamond ]
[   Obsidian    ][   Obsidian   ][   Obsidian    ]
```

Result: **1× Magic Crafting Table** (custom block, placed like a normal table)


The Magic Crafting Table GUI is a chest-like interface where you place materials in defined slots and press a "Craft" button (a Nether Star item in the output slot area).

---

### 7.3 Ritual Table

Crafted **inside the Magic Crafting Table**.

**Recipe (Magic Crafting Table GUI):**

```
[ Crying Obsidian ][ Magic Paper ][ Crying Obsidian ]
[     Quartz      ][ Quartz Slab ][     Quartz      ]
[ Crying Obsidian ][ Obsidian    ][ Crying Obsidian ]
```

Result: **1× Ritual Table**

---

### 7.4 Stardust Maker Table

Crafted **inside the Magic Crafting Table**. Used to convert runes and spell items into Stardust.

**Recipe (Magic Crafting Table GUI):**

```
[ Amethyst Shard ][ Amethyst Block ][ Amethyst Shard ]
[ Amethyst Shard ][ Smooth Basalt  ][ Amethyst Shard ]
[     Iron       ][  Iron  Ingot   ][     Iron       ]
```

Result: **1× Stardust Maker Table**

---

### 7.5 Stardust Enhancer Table

Crafted **inside the Magic Crafting Table**. Used to infuse Stardust into armor and tools.

**Recipe (Magic Crafting Table GUI):**

```
[ Gold Block ][ Amethyst Block ][ Gold Block ]
[ Gold Ingot ][ Smooth Basalt  ][ Gold Ingot ]
[ Iron Ingot ][   Iron Ingot   ][ Iron Ingot ]
```

Result: **1× Stardust Enhancer Table**

---

### 7.6 Magic Robes (in Magic Crafting Table)

See Section 9 for full details. All magic armor is crafted in the Magic Crafting Table using magic-type-specific materials.

---

## 8. Ritual Crafting System

### 8.1 How It Works

The **Ritual Table** is used exclusively for crafting **Spell items**. It cannot craft anything else.

**Required Ingredients per Spell (placed in the Ritual Table GUI):**

| Slot | Item Required |
|------|---------------|
| Center | 1× Magic Paper |
| Slot A | 1× Magic Rune (Type A — Element) |
| Slot B | 1× Effect Rune (Type B — Delivery) |
| Slot C | 1× Modifier Rune (Type C) |
| Slot D | 1× Level Rune (Type D) |

Click the **"Perform Ritual"** button (Nether Star in output). All 5 items are consumed and one **Spell Item** is produced, with a name automatically generated from the combination:

> Example: Ember Rune + Arrow Rune + Swift Rune + Adept Rune → **"Adept Swift Fire Bolt"**

### 8.2 Ritual Table GUI Layout

```
[ - ][ A ][ - ][ - ][ - ]
[ B ][ ★ ][ C ][ ⟹ ][ Spell Output ]
[ - ][ D ][ - ][ - ][ - ]
     [Magic Paper goes in ★ slot]
```

### 8.3 Spell Name Generation

Spell names are assembled automatically:
`[Level Rune Name] + [Modifier Rune Name] + [Element] + [Effect Name]`

Examples:
- Novice Wide Fire Nova → Tier I, large AoE fire explosion
- Master Pierce Dark Beam → Tier V, piercing dark ray (requires Dark Affinity + Power Level 12)
- Adept Echo Water Sigil → Tier III, double-casting water rune circle
- Expert Split Poison Arrow → Tier IV, arrow that splits into 3 poison bolts
- Apprentice Long Light Veil → Tier II, extended-duration healing aura (requires Light Affinity)

---

## 9. Magic Robes (Armor)

### 9.1 Overview

All magic robes are **Unbreakable** (cannot be damaged or repaired). They are equipped in the standard armor slots (helmet, chestplate, leggings, boots) and provide unique passive effects depending on the type. Each robe piece grants a **flat Mana Bonus** when worn.

### 9.2 Dyeable Magic Robe Set ("Arcane Robe")

The basic robe — no specific magic type, but **fully dyeable** using vanilla dye mechanics on leather armor (the robe uses leather armor as its base model).

**Stats (comparable to Iron Armor — full set):**

| Piece | Armor Points | Mana Bonus |
|-------|-------------|------------|
| Arcane Hood | 2 | +20 Mana |
| Arcane Robe | 5 | +30 Mana |
| Arcane Leggings | 4 | +25 Mana |
| Arcane Boots | 1 | +15 Mana |
| **Full Set Total** | **12** | **+90 Mana** |

**Set Bonus (wearing all 4 pieces):** +5% to all magic types effectiveness.

**Recipe (Magic Crafting Table, shapeless dyeable):**

Dye + 4× Magic Paper + 4× Leather → Corresponding robe piece

---

### 9.3 Magic-Type Specific Robe Sets

Each of the 7 magic types has its own robe set crafted in the Magic Crafting Table. All sets are **Unbreakable**. Stats differ per type to reflect their fantasy role. **Light and Dark sets require the player's active Affinity to match before they can be worn** — wearing the wrong type removes the passive bonuses and shows a warning.

---

#### 🔥 Fire Robe Set

**Passive Effects:**
- Full set: Fire Resistance (permanent)
- Chestplate: Fire spells deal +15% damage
- Boots: Leave fire trails when sprinting (briefly sets ground on fire)
- Helmet: Immunity to lava damage, see clearly inside lava

**Stats (Full Set):** 14 Armor Points | +100 Mana

**Recipe Materials:** Blaze Powder, Magma Cream, Nether Brick, Fire Charge, Magic Paper

---

#### 💧 Water Robe Set

**Passive Effects:**
- Full set: Water Breathing (permanent), no drowning
- Chestplate: Water spells restore 10% of damage dealt as health to the caster
- Boots: Walk on water surface (like Frost Walker, but thaws immediately after stepping off)
- Helmet: Night Vision while underwater

**Stats (Full Set):** 10 Armor Points | +120 Mana

**Recipe Materials:** Prismarine Shards, Heart of the Sea, Blue Ice, Kelp, Magic Paper

---

#### 🌿 Earth Robe Set

**Passive Effects:**
- Full set: Immunity to fall damage, Slow Falling effect always active
- Chestplate: +4 bonus armor points (stone-hard shielding)
- Boots: No crop trampling, completely silent footsteps
- Helmet: Night Vision in caves (below Y=40)

**Stats (Full Set):** 18 Armor Points | +80 Mana

**Recipe Materials:** Deepslate, Mossy Cobblestone, Emerald, Moss Block, Magic Paper

---

#### 💨 Air Robe Set

**Passive Effects:**
- Full set: +30% movement speed, immunity to knockback
- Chestplate: Air spells briefly stun struck enemies (0.5s)
- Boots: Double jump once per air-time (upward launch)
- Helmet: No fall damage from elytra crashes; glide is always available even without elytra equipped

**Stats (Full Set):** 8 Armor Points | +130 Mana

**Recipe Materials:** Feather, Phantom Membrane, Lightning Rod, White Stained Glass, Magic Paper

---

#### ☠ Poison Robe Set

**Passive Effects:**
- Full set: Immunity to Poison and Wither effects
- Chestplate: Poison spells stack an additional Wither I (2s) on poisoned targets
- Boots: Leave a brief poison cloud trail when sprinting (1-block radius, 2s duration)
- Helmet: See poisoned/withered mobs highlighted in green

**Stats (Full Set):** 11 Armor Points | +110 Mana

**Recipe Materials:** Spider Eye, Fermented Spider Eye, Wither Rose, Slimeball, Magic Paper

---

#### ✨ Holy / Light Robe Set *(Restricted — requires Light Affinity to wear)*

**Passive Effects:**
- Full set: Regeneration I (permanent, slower rate)
- Chestplate: Light spells heal the caster for 5% of damage dealt
- Boots: Undead mobs flee from the wearer within a 4-block radius
- Helmet: Permanent gentle glow effect; other players see a holy aura particle around the wearer

**Stats (Full Set):** 17 Armor Points | +130 Mana

> Higher mana bonus reflects the higher mana cost of Light spells.

**Recipe Materials:** Gold Ingot, Glowstone Dust, White Wool, Nether Star, Magic Paper

**Affinity Lock:** If a player with Dark Affinity equips a Light Robe piece, the piece's bonuses are suppressed and a warning appears: `"Your Affinity is not aligned with this robe."`

---

#### 🌑 Dark Robe Set *(Restricted — requires Dark Affinity to wear)*

**Passive Effects:**
- Full set: Mobs are less likely to target the wearer passively (stealth aura, 50% aggro reduction at range)
- Chestplate: Dark spells steal 5% of damage dealt as health (lifesteal)
- Boots: No step sounds at all; sprint makes the player nearly invisible in darkness (below light level 4)
- Helmet: See all living entities through walls within 6 blocks; detect hidden players on sneak

**Stats (Full Set):** 18 Armor Points | +130 Mana

> Higher mana bonus reflects the higher mana cost of Dark spells.

**Recipe Materials:** Coal Block, Ink Sac, Wither Rose, Obsidian, Magic Paper

**Affinity Lock:** If a player with Light Affinity equips a Dark Robe piece, the piece's bonuses are suppressed and a warning appears: `"Your Affinity is not aligned with this robe."`

---

## 10. Artifact System

### 10.1 Overview

Artifacts are **powerful magical relics** that cannot be crafted. They are found exclusively in world structures (shrines, magic houses, dungeons) or as boss mob drops. They provide strong passive bonuses and sometimes active secondary effects.

### 10.2 Artifact Equipment Slots

Artifacts are equipped in a **dedicated Artifact Menu** — a custom GUI opened via `/artifacts` or a bound key item. The menu has **6 slots**:

| Slot Name | Position in GUI | Example |
|-----------|----------------|---------|
| **Head Slot** | Top-center | Circlet of Stars |
| **Neck Slot** | Middle-center-top | Amulet of Flame |
| **Belly Slot** | Middle-center | Belt of the Deep |
| **Foot Slot** | Bottom-center | Boots of the Wind |
| **Left Hand Slot** | Middle-left | Void Ring |
| **Right Hand Slot** | Middle-right | Ember Bracelet |

Equipped artifacts are stored in player data (not in normal inventory) and their effects are always active.

### 10.3 Artifact GUI Layout

```
╔═══════════════════════════╗
║      [  Head Slot  ]      ║
║  [L.Hand] [Neck] [R.Hand] ║
║      [  Belly Slot ]      ║
║      [  Foot Slot  ]      ║
╚═══════════════════════════╝
```

### 10.4 Example Artifacts

| Artifact Name | Slot | Effect |
|---------------|------|--------|
| Circlet of Stars | Head | +50 Mana, +1 Mana/s regen |
| Amulet of Flame | Neck | Fire spells deal +20% damage |
| Eye of the Deep | Neck | Water Breathing, +30 Mana |
| Venom Choker | Neck | Poison spells apply Wither I for 3s extra |
| Belt of the Sage | Belly | +30 Mana, -10% all spell costs |
| Gale Runner Belt | Belly | +15% movement speed, Air spells cost -15% mana |
| Void Walker Soles | Foot | 10% chance to dodge projectiles |
| Gale Dancer Steps | Foot | +10% movement speed |
| Ember Bracelet | Hand | +15% Fire spell effectiveness |
| Thorn Ring | Hand | Poison spells deal +10% damage |
| Dawn Signet | Hand | Light spells +15% healing *(Light Affinity only)* |
| Dusk Signet | Hand | Dark spells +15% lifesteal *(Dark Affinity only)* |
| Ring of Sustenance | Hand | Regen 5 Mana per mob killed |
| Dual Signet | Hand | Works as both Dawn & Dusk Signet *(requires Dual Affinity)* |

---

## 11. Stardust System

### 11.1 What is Stardust?

**Stardust** is a special resource that represents the "crystallized magic" extracted from spell items and runes. It appears as a glowing dust particle item in the inventory. Stardust has **no crafting use by itself** — it is used only in the Stardust Enhancer Table to improve gear.

### 11.2 Stardust Maker Table

At the **Stardust Maker Table**, players can convert the following items into Stardust:

| Input | Stardust Gained |
|-------|----------------|
| Any Rune | 1 Stardust |
| Tier I Spell | 2 Stardust |
| Tier II Spell | 4 Stardust |
| Tier III Spell | 8 Stardust |
| Tier IV Spell | 15 Stardust |
| Tier V Spell | 25 Stardust |

> **Conversion is irreversible.** The input item is destroyed.

### 11.3 Stardust Enhancer Table — Armor

At the **Stardust Enhancer Table**, players can apply Stardust to **armor pieces** (both magic robes and vanilla armor) to increase their **Stardust Value (SV)**.

**How it works:**

- Place 1 armor piece + any amount of Stardust into the table.
- Click "Infuse".
- Each Stardust applied increases the SV of that armor piece by a **random roll** between **0.1% and 1.0%** (per Stardust unit).
- The SV is stored as a persistent item tag on the armor.
- The armor item displays its current SV in the lore: `✦ Stardust Value: 47.3%`

**SV Bonus Thresholds (Armor):**

| SV Reached | Bonus Unlocked |
|------------|----------------|
| 10% | +1 Armor Point |
| 25% | +2 Max Health (1 heart) |
| 50% | +2 Armor Points |
| 75% | +2 Max Health |
| 100% | +3 Armor Points, +2 Max Health, glowing particle effect |
| 150% | +1 to all magic type power levels (while wearing) |
| 200% | Unique visual aura + +5 Mana bonus per piece |

> Maximum SV is **200%**. Items above 200% are capped.

### 11.4 Stardust Enhancer Table — Tools

Vanilla tools (sword, pickaxe, axe, shovel, bow, etc.) can also be enhanced with Stardust.

**SV Bonus Thresholds (Tools):**

| SV Reached | Bonus Unlocked                                     |
|------------|----------------------------------------------------|
| 10% | +10% Durability                                    |
| 25% | +0.5 Attack Damage                                 |
| 50% | +15% Tool Speed (mining/attack)                    |
| 75% | +1 Attack Damage                                   |
| 100% | +25% Durability, +1 Damage, particle effect on hit |
| 150% | +10% Tool Speed                                    |
| 200% | +2 Attack Damage, permanent glow effect            |

---

## 12. World Structures

### 12.1 Overview

New structures generate naturally in the overworld and optionally in other dimensions. They are the primary source of **Runes** and **Artifacts**.

---

### 12.2 Magic Shrines

**Biome:** Any (tiered by biome rarity)
**Size:** Small (5×5×7 footprint)
**Frequency:** Moderate — roughly 1 per large chunk cluster

**Description:** A stone pillar structure with a glowing center altar block. The altar emits particles. Surrounding the pillar are carved rune markings on the floor (decorative).

**Contents:**
- 1 Locked Shrine Chest (requires player level 5+ to open)
- Loot: 1–3 random Runes, small chance of Stardust, very rare Artifact

**Variants:**

| Variant | Biome | Rune Loot Bias |
|---------|-------|---------------|
| Fire Shrine | Desert, Badlands, Nether | Ember Runes |
| Water Shrine | Ocean, Beach, River, Snowy Taiga | Tide Runes |
| Earth Shrine | Forest, Jungle, Dark Forest | Stone Runes |
| Air Shrine | Mountains, Plains, Savanna | Gale Runes |
| Poison Shrine | Swamp, Mangrove Swamp, Jungle | Venom Runes |
| Light Shrine | Sunflower Plains, Meadow, Cherry Grove | Dawn Runes *(rare)* |
| Dark Shrine | Deep Dark, Wither Cave, rarely in Dark Forest at night | Dusk Runes *(rare)* |

> Light and Dark Shrines are **significantly rarer** than standard type shrines, reflecting the restricted nature of their magic. Dark Shrines may also be guarded by stronger mob variants.

---

### 12.3 Magic Houses

**Biome:** Forest, Taiga, Plains
**Size:** Medium (9×9×7 building)
**Frequency:** Rare — roughly 1 per several hundred chunks

**Description:** A ruined stone-brick and wood cottage overgrown with vines and mushrooms. Inside is an abandoned ritual setup — candles, lecterns with magic books (decorative), and storage.

**Contents:**
- 2–3 Barrels / Chests with loot
- Loot: Runes (higher quantity than shrines), Magic Paper (1–3), Stardust (1–5), rare Spell item already crafted
- Possible rare: 1 Artifact (20% chance per Magic House)

---

### 12.4 Runic Dungeons

**Biome:** Underground (any)
**Size:** Large (custom dungeon layout, 3–5 rooms)
**Frequency:** Very Rare

**Description:** A hidden underground dungeon with magic traps (magic pressure plates that trigger debuff spells), mob spawners (custom magic-themed mobs), and a final room with a boss chest.

**Contents (Boss Chest):**
- 3–6 Runes (high tier, includes Level Runes IV–V)
- 1 guaranteed Artifact
- 5–15 Stardust
- 1–2 crafted Spell items

---

### 12.5 Runic Towers

**Biome:** Plains, Savanna
**Size:** Tall tower (5×5 base, ~20 blocks high)
**Frequency:** Uncommon

**Description:** A tall basalt/stone tower with a glowing top. Inside, a spiral staircase leads up to a small chamber with a chest. Guarded by Evoker-variants (Magic Guardians) near the top.

**Contents:**
- 1 Chest at top: 2–4 Runes, small Artifact chance, Magic Paper

---

## 13. UI & Menus Summary

| Menu | Open Method | Purpose |
|------|------------|---------|
| **Skill Menu** | `/skills` or Skill Tome item (right-click) | Spend Skill Points on upgrades |
| **Artifact Menu** | `/artifacts` or Artifact Codex item (right-click) | Equip/unequip Artifacts |
| **Magic Crafting Table** | Right-click placed Magic Crafting Table block | Craft robes, ritual table, stardust tables |
| **Ritual Table** | Right-click placed Ritual Table block | Craft Spell items from runes |
| **Stardust Maker** | Right-click Stardust Maker Table block | Convert runes/spells to Stardust |
| **Stardust Enhancer** | Right-click Stardust Enhancer Table block | Apply Stardust to armor and tools |

---

Here's the simplified version where God-Tier armor keeps the **exact same passive effects** as the base robe set, and **only the armor stats are doubled**:

---

## 14. ✦ God-Tier Magic Armor (Magic Star Enhancement)

Every **Magic-Type Specific Robe Set** (Fire, Water, Earth, Air, Poison, Light, and Dark) can be enhanced at the **Smithing Table** using a **Magic Star** to become a **God [Type] Magic Armor** — the strongest armor in the entire plugin.

> ⚠ The **dyeable Arcane Robe** set cannot be enhanced this way — only true magic-type robes can ascend to god-tier.

### How to Enhance

Use the vanilla **Smithing Table** to combine your magic robe piece with a **Magic Star**:

```
[ Magic Star ]  +  [ Magic-Type Robe Piece ]  →  [ God [Type] Magic Armor Piece ]
```


Each piece must be enhanced individually (4 Magic Stars to upgrade a full set).

### Crafting the Magic Star

The **Magic Star** is crafted in the **Vanilla Crafting Table** with one **Nether Star** surrounded by **8 Magic Paper**:

```
[ Magic Paper ][ Magic Paper ][ Magic Paper ]
[ Magic Paper ][ Nether Star ][ Magic Paper ]
[ Magic Paper ][ Magic Paper ][ Magic Paper ]
```


Result: **1× Magic Star**

> The Nether Star is consumed, making the Magic Star a high-value, late-game resource worthy of god-tier gear.

---

### God-Tier Stats

God-Tier armor keeps **all the same passive effects** as its base robe set — nothing about the type passives changes. The only difference is that the **Armor Points and Mana Bonus are doubled**.

All God-Tier armor remains **Unbreakable**.

#### Full Set Comparison (Base Robe → God-Tier)

| Magic Type | Base Armor | God Armor | Base Mana | God Mana |
|-----------|------------|-----------|-----------|----------|
| 🔥 **Fire** | 14         | **28**    | +100 | **+200** |
| 💧 **Water** | 10         | **20**    | +120 | **+240** |
| 🌿 **Earth** | 18         | **36**    | +80 | **+160** |
| 💨 **Air** | 8          | **16**    | +130 | **+260** |
| ☠ **Poison** | 11         | **22**    | +110 | **+220** |
| ✨ **Light** | 17         | **34**    | +130 | **+260** |
| 🌑 **Dark** | 18         | **36**    | +130 | **+260** |

> God-Tier armor points may exceed the vanilla 20-point cap — this is intentional. God-Tier wearers are meant to be exceptionally tanky, balanced by the extreme rarity of Nether Stars.

> ✨ **Light** and 🌑 **Dark** God-Tier armor still require the matching **active Affinity** to grant their bonuses (see Section 9.3 Affinity Lock). Ascending does not bypass the Affinity restriction.

---

## 15. Balance Notes

### Progression Flow (Intended Path)

```
New Player
    → Explore world, find first Shrine (standard type — Fire, Water, Earth, Air, Poison)
    → Collect Runes (low tier first)
    → Craft Magic Paper
    → Craft Magic Crafting Table
    → Craft Ritual Table
    → Craft first Tier I spell (using Novice Level Rune)
    → Use spells, earn XP, level up, spend SP in standard types
    → Upgrade preferred type to Power Level 3–5
    → Craft matching magic robe set
    → Find Stardust Maker and Enhancer, begin infusing armor
    → Encounter a Light Shrine or Dark Shrine
    → Spend 5 SP to unlock Light or Dark Affinity (major decision moment)
    → Invest in Light or Dark tree, use restricted spells
    → Farm higher-tier runes, craft Tier IV–V spells
    → Explore runic dungeons for artifacts
    → Consider spending 60 SP for Dual Affinity (very late-game)
    → Reach Power Level 10 in multiple types, build final loadout
```

### Key Constraints

- **Artifacts** are never craftable — always world-found.
- **Magic Robes** are unbreakable; no repair cost ever.
- **Light and Dark Robes** are suppressed (no bonuses) if the wrong Affinity is active, but the items are not destroyed.
- **Stardust** is a slow incremental system — full 200% SV requires significant investment.
- **Spells** require both the correct Magic Power Level AND sufficient Mana. High-tier Light/Dark spells are especially expensive.
- **Skill Points** are finite but can be reset via a **Tome of Respec** (craftable in Magic Crafting Table from Magic Paper + Lapis Block). Resetting **does not refund** the 5 SP Affinity Unlock gate cost for Light or Dark — once unlocked, it stays unlocked. Affinity Switch ranks **are** refundable via respec.
- **Affinity Switch Rank 3** once purchased grants simultaneous use permanently, but is refunded by respec (the cooldown returns to whatever rank remains after the refund).
- The **Affinity switch cooldown** persists through relog and server restart (stored in player data).
- **Scaling costs** mean that deeply investing in any single node is a major commitment. Players cannot max every tree — they must specialize.
- Magic armor or god maguic armor is also enhancable with stardust

### Anti-Abuse Notes

- Ritual Table should prevent spam-crafting (short cooldown between rituals: 2 seconds).
- Stardust rolls should be server-side to prevent manipulation.
- Light/Dark Affinity switching should be logged server-side to detect exploit attempts.
- Dawn Rune and Dusk Rune should have their crafting Affinity validated **server-side** at click time — not just client-side display.
- Scaling SP costs should be calculated server-side on purchase confirmation, not trusted from client input.

---

*Document End — ArcaneCore v0.1 Design Draft*
