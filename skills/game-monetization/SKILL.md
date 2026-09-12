---
name: game-monetization
last_reviewed: 2026-09-06
group: Craft
description: >-
  Design free-to-play economies, battle passes and cosmetic shops with anti-predatory guardrails.
  Use when designing battle passes, gacha, in-game economies, or IAPs.
---

# game-monetization

## Core Philosophy
Video game monetization is not tricking players with manipulative dark patterns, predatory gacha mechanics, or pay-to-win combat advantages that destroy competitive integrity. Sustainable, high-LTV game monetization is built on mutual respect, long-term retention, authentic player celebration, and vanity-first cosmetics. Whether designing free-to-play (F2P) battle passes, premium DLC expansions, or community creator economies, monetization must enhance the player experience rather than hold it hostage.

---

## 4-Step Sustainable Game Monetization Architecture

### Step 1: Monetization Model Selection & Guardrails
1. **The 3 Ethical Monetization Models**:
   - *Premium Upfront ($20–$70)*: Full game unlocked; optional paid story DLC expansions.
   - *Free-to-Play Cosmetic / Battle Pass*: Core competitive gameplay is 100% free and fair; monetization driven strictly by cosmetics, emotes, character skins, and audio packs.
   - *Subscription / Season Pass*: Continuous content delivery for dedicated enthusiasts (e.g. MMOs).
2. **The Non-Negotiable Anti-Predatory Guardrails**:
   - **Zero Pay-to-Win (P2W)**: Money must never buy statistical power, faster cooldowns, or competitive advantages in PvP.
   - **Probability Transparency**: If randomized cosmetic drops (loot boxes) are used, exact drop probabilities must be displayed mathematically on screen down to 0.01% (enforcing China/EU statutory mandates).
   - **Spending Limits & Minor Protection**: Enforce spending velocity caps and parental verification gates.

### Step 2: Battle Pass Progression Architecture
1. **Dual-Track Progression (Free vs Premium)**:
   - *Free Track*: Grants enough currency, consumables, and modest skins to make non-paying players feel valued and engaged.
   - *Premium Track ($10 / ~1,000 Hard Currency)*: Grants exclusive seasonal skins, custom animations, weapon charms, and enough earned hard currency to fund the *next* season's pass if completed.
2. **Linear vs Non-Linear Progression Curves**:
   - Avoid aggressive grind walls: Target **50–70 hours** of casual gameplay across a 90-day season to complete 100 tiers.
   - Provide daily and weekly gameplay challenges that reward XP without dictating frustrating, disruptive playstyles.

### Step 3: Virtual Economy & Currency Sinks
1. **The Two-Currency System**:
   - *Soft Currency (Earned Gameplay Only)*: Gold / Credits earned by playing matches. Used for baseline character upgrades, weapon repairs, and basic customization.
   - *Hard Currency (Purchased with Real Cash)*: Gems / Coins purchased via storefront. Used exclusively for premium cosmetics and battle passes.
2. **Inflation Control & Sinks**:
   - Soft currency sinks (crafting fees, cosmetic unlocks) must mathematically match soft currency generation rates to prevent hyperinflation.

### Step 4: The Rotating Cosmetic Shop (FOMO vs Respect)
1. **Rotating Storefront Discipline**:
   - Feature a rotating catalog of curated cosmetic bundles.
   - Provide clear direct-purchase options (players can buy the exact item they want directly without buying 20 mystery boxes).
   - Maintain evergreen availability for signature legacy items to minimize toxic FOMO (Fear of Missing Out).

---

## Deliverable Format: Game Economy & Monetization Specification (`MONETIZATION-SPEC.md`)

```markdown
# Economy & Monetization System Specification: [Game Title]

## 1. Core Model & Ethical Guarantees
- **Model**: Free-to-Play Multiplatform Tactical Shooter
- **P2W Policy**: 100% Cosmetic Only (Zero stat advantages, zero paywalled weapons)
- **Primary Revenue Levers**: Seasonal Battle Pass ($10) + Direct Cosmetic Storefront

## 2. Currency Architecture
| Currency | Type | Source | Primary Sinks | Inflation Safeguard |
|---|---|---|---|---|
| Credits | Soft Currency | Match completion, daily quests | Base gun recolors, profile banners | Fixed repair fees per match |
| Stellar Gems| Hard Currency | Real-money purchase ($1 = 100 Gems) | Battle Pass, Legendary Hero Skins | Fixed direct prices |

## 3. Seasonal Battle Pass Progression (90-Day Season)
- **Tier Count**: 100 Tiers (Free Track: 30 rewards | Premium: 100 rewards)
- **Completion Horizon**: ~60 hours of gameplay (Avg 45 mins/day)
- **Currency Recirculation**: Completing the Premium Pass earns 1,000 Stellar Gems (100% cost of next pass).

## 4. Storefront Pricing & Transparency
- Common Skin: 300 Gems ($3.00)
- Rare Skin: 800 Gems ($8.00)
- Legendary Hero Bundle: 1,800 Gems ($18.00)
- *All items available via direct purchase; zero randomized loot boxes.*
```

---

## Worked Example: Turning Around Player Revolt Over Aggressive Monetization

- **Crisis**: Players revolted when a shooter introduced a paid character that had 10% faster health regeneration. Steam reviews tanked to "Mostly Negative".
- **Remediation**: Leadership immediately removed all combat stats from the character; refunded all purchases in hard currency; issued a public apology pledging a permanent "Cosmetics-Only" charter.
- **Outcome**: Community trust restored; long-term Battle Pass retention increased 45% over subsequent seasons.

---

## Verification Checklist

- [ ] Zero competitive combat advantages or pay-to-win items sold for cash.
- [ ] Premium Battle Pass returns sufficient hard currency to purchase the next season.
- [ ] Progression curve allows casual players to finish tiers within season limits (50–70 hrs).
- [ ] Soft currency generation is balanced with durable economic sinks.
- [ ] Direct purchase is supported for all featured cosmetic assets.

---

## Anti-Patterns

- **Pay-to-Win Sneakiness**: Selling weapons that are statistically superior to free weapons.
- **Energy / Paywall Chokeholds**: Blocking players from playing matches after 30 minutes unless they pay "energy refills".
- **Predatory Probability Manipulation**: Secretly altering drop chances to force whales into spending thousands on unannounced odds.
