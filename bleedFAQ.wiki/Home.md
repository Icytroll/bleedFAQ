# Bleeds 101

In Diablo 4 a bleed can be described as a physical damage over time effect that usually lasts for 5 seconds on an enemy, meaning physical and damage over time bonuses are always included when a bleed is applied to an enemy.

A few quick facts about bleeds:

- Lasts for 5s unless the the bleed source specifically states otherwise
- Bleed damage is distributed into portions that are dealt every 0.5s, also called ticks
- Unlike direct damage, bleed damage cannot critically strike (unless we have Gushing Wounds, more on that later) or Overpower

[D4 discord - damage calculations](https://discord.com/channels/989899054815281243/1213607031169224714/1213607031169224714)

## Snapshotting

Test 123 456

## 2H sword expertise

The Barbarian's weapon arsenal also holds

## Berserk Ripping

The Berserk Ripping aspect in a special case of bleed for barbarian as the bleed damaged scales with the **base** damage of any direct damage you deal while berserking, and is not a fixed amount determined by a skill percentage via e.g. Flay or Rend. This can create some very powerful combinations if we can find interactions with very large direct hits that count as base damage.

![Gushing Wounds tooltip](images/BR.png)

These **DO** work

- Elements aspect (phys buff)
- Inner Calm aspect
- Retribution aspect
- Paingorger's unique
- Bash Cleave tempers

These **DO NOT** work

- Aspect of Adaptability
- Aspect of the Moonrise
- Edgemaster's aspect
- Ramalamadingdong

## Gushing Wounds

Gushing Wounds is a Barbarian Key Passive that greatly augments our bleeds by letting them interact with our Critical Strike Chance (CHC) and Critical Strike Damage (CSD). The ability tooltip can be seen below, however I'd like to rewrite that description with how it actually works in-game:

**_When causing an enemy to bleed, you have a chance equal to [the "Critical Strike Chance" stat in your statsheet] to increase the bleed by an amount equal to [the "Critical Strike Damage" stat in your statsheet]._**

![Gushing Wounds tooltip](images/GW.png)

In short, Gushing Wounds directly uses the numbers in the "Critical Strike Chance" and "Critical Strike Damage" categories from the stat sheet to calculate if a bleed crits and how big that crit is going to be. For example, in the below image a bleed would have a 67.7% chance to have its total bleed amount increased by 3672.6%.

![Gushing Wounds tooltip](images/statsheetCrit.png)

This is very unusual because almost every single ability in the game does **not** use the up-front stat sheet values to calculate their values, but the **raw** amounts gained from gear and paragon boards (green box in the image below). You might ask, well aren't the immediatetly up-front values correct? Yes and no, though more no than yes.

The magenta value is an attempt to incorporate all the critical damage effects that are currently active on a player, both additive CSD that you see with a (+), as well as multiplicative effects that are shown with a (x). The total CSD number in magenta is calculated as follows:

```
totalStatSheetCSD = (1+rawCSD%)*(1+50%)*(1+HH%)*(1+GF%) - 1
```

where `rawCSD%` is the number shown in the green box, `HH%` is the multiplicative amount gained via the Heavy Handed skill passive, and `GF%` is the x100% CSD bonus gained from equipping the über unique sword, The Grandfather. Though there are more than just Heavy Handed and Grandfather that provides a multiplicative CSD bonus, for barbarians they are the only ones that affect the statsheet CSD, meaning they are the only ones to affect how much bleeds are scaled by when they crit via Gushing Wounds.

One thing to be clear about with the total statsheet CSD is that it tells you the amount of damage you will do when you crit, not the amount that is added onto a regular attack when critting. Not only can this be slightly unintuitive depending on how one thinks about critical strikes in general, but it's certainly misleading in the sense that this value is described as "Extra damage granted to Skills when they Critically Strike", which is absolutely not the general case when a crit is calculated. I won't go too far into this here, just know that you should **never** assume that the magenta value is being used by the game for calculations, it's merely an (incorrect) attempt by Blizzard to show the player how hard their crits hit for. **Always** use the raw CSD value (green) to do calculations with.

![Gushing Wounds tooltip](images/rawCrit.png)

With all that being said, Gushing Wounds is a rare exception to this pattern of the game using raw values for calculations. When using it as a crit multiplier in damage calculations for bleeds we simply write:

```
GWmult = 1 + totalStatSheetCSD*140%
```

and if we wanted to rewrite this to using the raw CSD% from gear and paragon for consistencies sake, we get

```
GWmult = 1 + ( (1+rawCSD%)*(1+50%)*(1+HH%)*(1+GF%) - 1 ) * 140%
       = 2.1 * (1+rawCSD%)*(1+HH%)*(1+GF%) - 0.4
```

For the case where no crit damage multipliers are present the above simplifies to

```
GWmult = 2.1*rawCSD% + 1.7
```

This means that if we for example have 1000% total additive damage and 820.4% raw CSD with no additional CSD multipliers, a bleed that crit via Gushing Wounds will have a combined additive\*crit multiplier of

```
critBleed = (1 + 1000%) * (2.1*820.4% + 1.7)
          = 11 * 18.93
          = 208.2
```

This is quite a bit more powerful compared to direct hits which only scale off of a x1.5 multiplier, if we were to use the same values for a direct hit we'd get a combined additive\*crit multiplier of

```
critHit = (1 + 1000% + 820.4%) * 1.5
        = 19.2 * 1.5
        = 28.8
```

So a bleed in the above example would have a combined additive\*crit multiplier that is ~7 times larger than that of a direct hit. This is of course not the whole story, we'd have to take other factors like skill damage, strength, aspects and a myriad of other multipliers into account for a complete picture. Just know that Gushing Wounds makes CSD quite attractive for bleed builds as it essentially moves the CSD out of the additive bucket and makes it a separate multiplier.
