# Detect when a mob has died

This article applies to Java Edition only. For Bedrock Edition the method is described in this wiki article: [Detect the player killing entities and players](/questions/playerkills.md#bedrock)

When a mob begins to play the death animation, then this mob cannot be selected using the target selector, this is the main difficulty in detecting the death of a mob, but there are still several ways to do this, but each has its own limitations. Each of the methods listed below is completely resistant to unloading mobs from loaded chunks and there will be no false positives.

----

## Scoreboard

The easiest way to implement is to use [scoreboard criteria](https://minecraft.wiki/w/Statistics#Statistic_types_and_names) `killed:<entity>`, but provides minimal opportunities for detection.

Key points:
* low server load
* can detect mob death only if the killer is a player
* cannot determine the location of a mob's death
* cannot detect the death of a mob with the tag
* each mob type must have a separate scoreboard objective.

Here's a quick example to detect if a zombie was killed by a player:

    # In chat / load function
    scoreboard objectives add killed.zombie killed:zombie
    
    # Command blocks / tick function
    execute as @a[scores={killed.zombie=1..}] run say Some zombie has died.
    scoreboard players reset @a[scores={killed.zombie=1..}] killed.zombie

----

## player_killed_entity advancement trigger

The implementation using `player_killed_entity` [advancement trigger](https://minecraft.wiki/w/Advancement/JSON_format#minecraft:player_killed_entity) has a little more possibilities.

Key points:
* minimal load on the server
* cannot determine the location of a mob's death
* you can check any number of entity types in one advancement
* can check any data of a mob that died, including tags, score, etc.
* can detect mob death only if the killer is a player
* requires using a datapack

Here's a quick example to detect if a any skeleton type with the tag some\_tag was killed by a player:

    # advancement example:killed_skeleton
    {
      "criteria": {
        "requirement": {
          "trigger": "minecraft:player_killed_entity",
          "conditions": {
            "entity": [
              {
                "condition": "minecraft:entity_properties",
                "entity": "this",
                "predicate": {
                  "type": "#minecraft:skeletons",
                  "nbt": "{Tags:['some_tag']}"
                }
              }
            ]
          }
        }
      },
      "rewards": {
        "function": "example:skeleton_death"
      }
    }
    
    # function example:skeleton_death
    advancement revoke @s only example:killed_skeleton
    say Skeleton with tag some_tag has dead.

----

## Passenger entity

This method is a direct way to check the death of a specific mob with any cause of death, not just due to the player.

This method is based on using a [marker entity](https://minecraft.wiki/w/Marker) as a passenger on a mob that needs to be checked for death. The passenger entity will ride the entity until the end of the death animation, so using this you can check the `DeathTime` tag of a mob with a death animation. This way also has a unique opportunity to check not only the death of a mob, but also when and where the mob despawned.

Key points:
* average load on the server
* this requires using a datapack (for versions 1.16 - 1.19.3)
* can check any data of a mob that died, including tags, score, etc.
* preliminary mob setup is required
* can determine the location of a mob’s death
* mob death can be detected with any cause of death
* does not work for mobs that already have a passenger
* can check mob despawn

This method requires preliminary setup of the mob, for example, you can summon a mob with an entity marker as a passenger:

    summon husk ~ ~ ~ {Tags:["some_tag"],Passengers:[{id:"minecraft:marker",Tags:["death_detector"]}]}

From version 1.19.4 you can also add a marker entity as passenger to an existing mob use [/ride command](https://minecraft.wiki/w/Commands/ride):

    summon marker ~ ~ ~ {Tags:["death_detector","set_ride"]}
    ride @e[type=marker,tag=death_detector,tag=set_ride,limit=1] mount <mob>
    tag @e[type=marker,tag=set_ride] remove set_ride

On versions 1.16 - 1.19.3, the only way to check this mob is to use the [predicate](https://minecraft.wiki/w/Predicate) in the datapack:

    # function example:tick
    execute as @e[type=marker,tag=death_detector,predicate=example:death_mob] run function example:death_mob
    execute as @e[type=marker,tag=death_detector,predicate=example:despawn_mob] run function example:despawn_mob
    
    # function example:death_mob
    say Mob is dead!
    kill @s
    
    # function example:despawn_mob
    say Mob is despawn!
    kill @s
    
    # predicate example:death_mob
    {
      "condition": "minecraft:entity_properties",
      "entity": "this",
      "predicate": {
        "vehicle": {
          "nbt": "{DeathTime:1s}"
        }
      }
    }
    
    # predicate example:despawn_mob
    {
      "condition": "minecraft:inverted",
      "term": {
        "condition": "minecraft:entity_properties",
        "entity": "this",
        "predicate": {
          "vehicle": {}
        }
      }
    }

As of version 1.19.4, you can use the [execute](https://minecraft.wiki/w/Commands/execute#on) `on vehicle` subcommand to select a dying mob:

    # Command blocks
    execute as @e[type=marker,tag=death_detector] on vehicle unless data entity @s {DeathTime:0s} run say This mob is dying!
    execute as @e[type=marker,tag=death_detector] on vehicle unless data entity @s {DeathTime:0s} on passengers run kill @s

However, this implementation on command blocks may cause lags due to NBT checks every tick, so it is recommended to use a [check delay](/questions/blockdelay.md) or use a datapack with [schedule function](https://minecraft.wiki/w/Commands/schedule):

    # function example:load
    function example:loops/1s
    
    # function example:loops/1s
    schedule function example:loops/1s 1s
    execute as @e[type=marker,tag=death_detector] on vehicle unless data entity @s {DeathTime:0s} run function example:death_mob
    execute as @e[type=marker,tag=death_detector,predicate=example:despawn_mob] run function example:despawn_mob
    
    # function example:death_mob
    say Mob is dead!
    execute on passengers run kill @s
    
    # function example:despawn_mob
    say Mob is despawn!
    kill @s

----

## Reading data by UUID (1.20.2+)

This is the most accurate way to check for mob death, since you access the mob's data directly using the [UUID](https://minecraft.wiki/w/Universally_unique_identifier), but it is also the most complex to implement overall.

_Without using a datapack, you need to manually enter the mob's UUID into each command, which makes this method practically useless when using only command blocks._

Key points:
* high load on the server
* maximum test accuracy
* direct data check
* requires datapack + [datapack UUID converter library](https://github.com/gibbsly/gu)
* high complexity of implementation
* cannot check mob despawn

The difficulty with this method is that need to get the mob's UUID in Hexadecimal format (`ba7916ab-abc0-4ef6-8a43-391dbffdefcd`), but data get can only get the UUID as a list of int values (`[I;-1166469461,-1413460234,-1975305955,-1073877043]`). Therefore, need to use a UUID converter. In this article will use this [UUID converter library](https://github.com/gibbsly/gu) by u/gibbsly.

To convert UUID using this library you need to run the `function gu:convert` as a macro function with the data of the selected mob, and then read the `storage gu:main out` tag.

In addition, you need to save the converted UUIDs of mobs in storage and use a macro to insert these UUIDs into your check command.

After detecting the death of a mob, it is also necessary to remove the UUID from storage, since macro functions can quickly cause lags with a large number of checks. For the same reason, you need to use a [delay for checks](/questions/blockdelay.md).

Below is an example to check the death of any mob that has the `death_check` tag:

    # function example:load
    function example:loops/1s
    
    # function example:loops/1s
    schedule function example:loops/1s 1s
    execute as @e[tag=death_check,tag=!init] run function example:death_check/init
    data modify storage example:macro death_check set from storage example:database death_check
    function example:death_check/check with storage example:macro death_check[-1]
    
    # function example:death_check/init
    function gu:convert with entity @s
    data remove storage example:data this
    data modify storage example:data this.UUID set from storage gu:main out
    data modify storage example:database death_check append from storage example:data this
    data modify entity @s PersistenceRequired set value true
    tag @s add init
    
    # function example:death_check/check
    $execute as $(UUID) unless data entity @s {DeathTime:0s} at @s run function example:death_check/death_event {UUID:$(UUID)}
    $data remove storage example:macro death_check[{UUID:$(UUID)}]
    function example:death_check/check with storage example:macro death_check[-1]
    
    # function example:death_check/death_event
    $data remove storage example:database death_check[{UUID:$(UUID)}]
    say This mob is dying!
    particle minecraft:soul_fire_flame ~ ~1 ~ 0.5 1 0.5 0.01 100

**Note:** This method cannot detect mob despawn, so you need to set the `PersistenceRequired:true` tag to prevent despawn.

You can use the example above as is. When a mob dies, the function `example:death_check/death_event` will be run in which the first command should remove the mob's UUID from storage, and the rest you can change the commands as you want.
