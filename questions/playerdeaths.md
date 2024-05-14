# Detect Player Death

_Related: [Detect Player Kills](/wiki/questions/playerkills)_

## Java

### Just death check

In Java, detecting a dead player is relatively easy. For simple death detection you can use [`deathCount`](https://minecraft.wiki/w/Scoreboard#Single_criteria) / [`custom:deaths`](https://minecraft.wiki/w/Statistics#List_of_custom_statistic_names) scoreboard criteria and whenever that one increases, the player just died (and is likely still dead).

    # In chat / load function
    scoreboard objectives add death deathCount
    
    # Command blocks / tick function
    execute as @a[scores={death=1..}] run say Death!
    scoreboard players reset @a death

### Run command at death position

To do this you can use the method described above, but if gamerule `doImmediateRespawn` is true, then the commands from the example above will actually be executed after respawn, which can be a problem if you want to do something at the player's death position.

If you are limited to using command blocks, then you can read the [player data](https://minecraft.wiki/w/Player.dat_format) `LastDeathLocation` tag (1.19+) to get dimension (`dimension`) and position (`pos`). But since the `LastDeathLocation.pos` tag is an Int Array, but the Pos tag entity is a list, you need to first convert the Int Array to a Double list. Then check the `LastDeathLocation.dimension` in which dimension the player died and set this dimension [`execute in <dimension>`](https://minecraft.wiki/w/Commands/execute#in), summon area_effect_cloud and move to the death position and in the same tick execute the command on the position of this area_effect_cloud entity.

Below is an example for 1.19.4 and above versions:

    # Setup
    data merge storage example:data {pos:{list:[0d,0d,0d],int_array:[I;0,0,0]}}

    # Command blocks
    data modify storage example:data pos.int_array set from entity @a[scores={death=1..},limit=1] LastDeathLocation.pos
    execute store result storage example:data pos.list[0] double 1 run data get storage example:data pos.int_array[0]
    execute store result storage example:data pos.list[1] double 1 run data get storage example:data pos.int_array[1]
    execute store result storage example:data pos.list[2] double 1 run data get storage example:data pos.int_array[2]
    execute if data entity @a[scores={death=1..},limit=1] LastDeathLocation{dimension:"minecraft:overworld"} in minecraft:overworld summon area_effect_cloud store success score @s death run data modify entity @s Pos set from storage example:data pos.list
    ...
    execute at @e[type=area_effect_cloud,scores={death=1}] run summon zombie ~ ~ ~ {PersistenceRequired:true,CanPickUpLoot:true}
    scoreboard players reset @a[scores={death=1..},limit=1] death

This method requires reading data for each dimension in the world in a separate command block.

Using a datapack will make executing any command in the death position much easier. This method is based on the fact that the [advancement trigger](https://minecraft.wiki/w/Advancement/JSON_format) `minecraft:entity_hurt_player` does not depend on the tick schedule, but on events, so this trigger is executed before the player is respawned, and even before the scoreboard is updated, so score health will not work, but only the NBT check player data:

    # advancement example:death
    {
      "criteria": {
        "requirement": {
          "trigger": "minecraft:entity_hurt_player",
          "conditions": {
            "player": [
              {
                "condition": "minecraft:entity_properties",
                "entity": "this",
                "predicate": {
                  "nbt": "{Health:0f}"
                }
              }
            ]
          }
        }
      },
      "rewards": {
        "function": "example:death"
      }
    }
    
    # function example:death
    advancement revoke @s only example:death
    summon zombie ~ ~ ~ {PersistenceRequired:true,CanPickUpLoot:true}

Likewise, using `minecraft.custom:minecraft.time_since_death` you can detect a player who just respawned, since this objective will stay 0 during the death screen and start counting up the moment the player clicks "Respawn".

    # Setup
    scoreboard objectives add respawn custom:time_since_death
    
    # Command blocks / tick function
    execute as @a[scores={respawn=1}] run say I just respawned!

## Bedrock

In Bedrock the same question is more difficult to answer, since at the time of writing only the `dummy` objective type exists.

The best way to do this in bedrock is with the following commands, in this order:

    /tag @a add dead
    /tag @e[type=player] remove dead
    /scoreboard players add @a[tag=dead,tag=!still_dead] deathCount 1
    /tag @a add still_dead
    /tag @e[type=player] remove still_dead

Set up a dummy scoreboard called `deathCount` and it will count up every time a player dies.  

This works because the `@a` selector selects all players, but the `@e` selector can only select living entities.

_This system was first suggested on the subreddit by /u/Sprunkles137 [here](https://old.reddit.com/r/MinecraftCommands/comments/g5b4n8/challenge_1/fo3p5p0/)._

There is a different way as well, which can only be used in a very controlled environment where players cannot set their own spawnpoints with beds but all spawning is controlled by the mapmaker. This way you can set everyone's spawnpoint to an otherwise inaccessible position in the world, then detect players respawning there, count up their death score and teleport them to the "actual" spawnpoint. **It is not advised to use this system anymore, as the first system is easier to do, requires less setup and has a wider range of applicable possibilities!**
