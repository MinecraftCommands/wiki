# Make a mob attack another specfic entity / player

The basic principle for this method is that a mob that can attack will get angry at whoever attacks them. _(some exceptions apply)_

## Java (without /damage)

### Spawning a projectile with a specific owner

The way the game knows who got attacked by whom is by tracking the owner of a projectile through the projectiles `Owner` tag, which represents the owners UUID. We can use this to make a projectile that pretends to be from a specific entity by modifying this NBT tag.

For example you can use a snowball, as this projectile only deal damage to blazes. The snowball is summoned 2.3 blocks above the target so their hitboxes don't overlap (which would cause the snowball to ignore the target). 
> [!IMPORTANT]
> If the player or entity is crowling or has a block above them, it wont work.

In the following example a skeleton tagged `attacker` is tricked into attacking a zombie tagged `target`.

    # summon the snowball
    execute at @e[type=skeleton,tag=attacker] run summon minecraft:snowball ~ ~2.3 ~ {Tags:["atk_target"]}
    # make the snowball owner the zombie
    execute as @e[type=minecraft:snowball,tag=atk_target] run data modify entity @s Owner set from entity @e[type=zombie,limit=1,tag=target] UUID


## Bedrock and Java

### Using /damage

_Info: In bedrock a lot of entities have a very defined set of other entities they can attack, as defined in their behavior pack. So if the below method doesn't work to anger them, make sure they're actually able to attack the target entity and change the behavior files accordingly!_

Thanks to the introduction of the [`/damage` command](https://minecraft.wiki/wiki/Commands/damage) in 1.18.10 we can use this command to inflict fake damage from one entity onto another with relative ease:

    # bedrock
    /damage <target> <amount> entity_attack entity <source entity>
    # java
    /damage <target> <amount> <damage_type> by <source entity>

so for example, to make an (untamed) wolf attack a player, you can run this command

    # bedrock
    /damage @e[type=wolf] 0 entity_attack entity @p
    # java
    /damage @e[type=wolf] 0 player_attack by @p

or to make a skeleton attack a zombie

    # bedrock
    /damage @e[type=skeleton] 0 entity_attack entity @e[type=zombie]
    # java
    /damage @e[type=skeleton] 0 player_attack by @e[type=zombie,limit=1]

In this example we're using `0` as the amount of damage, as we just want to pretend to deal damage to the entity, not actually deal any damage.

## Bedrock

### Using behaviors

As mentioned above, what entities can and cannot be attacked by another entity is governed by the entities behavior file (in the `minecraft:behavior.nearest_attackable_target` component). For example the skeleton is actively seeking out players, iron golems and baby turtles on land as their targets to attack without provocation. This component can be modified to automatically attack other entities, too.

Here is the relevant component in the vanilla skeleton behavior file (1.17):

      "minecraft:behavior.nearest_attackable_target": {
        "priority": 2,
        "must_see": true,
        "reselect_targets": true,
        "entity_types": [
          {
            "filters": {
              "test": "is_family",
              "subject": "other",
              "value": "player"
            },
            "max_dist": 16
          },
          {
            "filters": {
              "test": "is_family",
              "subject": "other",
              "value": "irongolem"
            },
            "max_dist": 16
          },
          {
            "filters": {
              "all_of": [
                {
                  "test": "is_family",
                  "subject": "other",
                  "value": "baby_turtle"
                },
                {
                  "test": "in_water",
                  "subject": "other",
                  "operator": "!=",
                  "value": true
                }
              ]
            },
            "max_dist": 16
          }
        ]
      },
