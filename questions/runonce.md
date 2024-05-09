# Activate a command *once* when a player does something (e.g: enters an area)

This makes a command act as if it was on a comparator, without the lag and multiplayer incompatibility that comes from using a comparator. The general idea here is to select players that match a selector, but did **not** match that same selector in the previous tick. For example, players who have just entered an area (with `@a[x=73,y=10,z=3,distance=..1]`), just gained level 5 (with `@a[level=5]`), just entered creative (with `@a[gamemode=creative]`), etc.  

## Scoreboard

This way is that the first command checks the player whose score value, for example, `matched` = 0, and if your any condition, for example, the player is in a certain area, is met, then execute your command that you want. But the second command selects all players and store the success of your condition in score (`matched`), which you checked in the first command:

    # In chat / load function
    scoreboard objectives add matched dummy
    
    # Command blocks / tick function
    execute as @a[scores={matched=0},x=73,y=10,z=3,distance=..1] run say I just entered the area!
    execute as @a store success score @s matched if entity @s[x=73,y=10,z=3,distance=..1]

**Note:** The order in which the commands are executed is important here. In the first command you check your condition and execute the command, and the second command store the success of executing your condition.

Besides the player, this could be some kind of global event, for example, it started to rain (1.20.5+):

    # Command blocks / tick function
    execute if score #raining matched matches 0 if predicate {condition:"weather_check",raining:true} run say It's starting to rain!
    execute store result score #raining matched if predicate {condition:"weather_check",raining:true}

## Advancements

If you are using a datapack and need to do a lot of similar checks, then you can use [advancements](https://minecraft.wiki/w/Advancement/JSON_format) in the datapack for this.

This method is to create a [predicate](https://minecraft.wiki/w/Predicate) that you check for the player, for example, the player is on server spawn (`predicate example:at_spawn`), and create two advancements - the first one checks the predicate - if the player is at spawn, and the second one which inverts this check (player leave spawn). Then inside the run function you execute the desired command when the player enters / leaves spawn and revoke just the opposite of the advancement.

    # predicate example:at_spawn
    {
      "condition": "minecraft:location_check",
      "predicate": {
        "position": {
          "x": {
            "min": -100,
            "max": 100
          },
          "z": {
            "min": 200,
            "max": 400
          }
        }
      }
    }
    
    # advancement example:spawn/enter
    {
      "criteria": {
        "requirement": {
          "trigger": "minecraft:location",
          "conditions": {
            "player": [
              {
                "condition": "minecraft:reference",
                "name": "example:at_spawn"
              }
            ]
          }
        }
      },
      "rewards": {
        "function": "example:spawn/enter"
      }
    }
    
    # function example:spawn/enter
    advancement revoke @s only example:spawn/leave
    tellraw @s "Welcome to spawn!"
    
    # advancement example:spawn/leave
    {
      "criteria": {
        "requirement": {
          "trigger": "minecraft:location",
          "conditions": {
            "player": [
              {
                "condition": "minecraft:inverted",
                "term": {
                  "condition": "minecraft:reference",
                  "name": "example:at_spawn"
                }
              }
            ]
          }
        }
      },
      "rewards": {
        "function": "example:spawn/leave"
      }
    }
    
    # function example:spawn/leave
    advancement revoke @s only example:spawn/enter
    tellraw @s "Welcome to spawn!"

Such a check may seem very large, but this method allows you not to check the same condition 2 times per tick, but only 1 time per second, which can be important with a large online number of players.

## Add/remove tag

The following commands, running in this order, will keep track of whether a player matched the selector `@a[x=73,y=10,z=3,distance=..1]`:

    tag @a[tag=alreadyMatched] remove alreadyMatched
    tag @a[x=73,y=10,z=3,distance=..1] add alreadyMatched

Players who matched the selector will get the `alreadyMatched` scoreboard tag (you can call this scoreboard tag whatever you want, so long as you're consistent with it). At the start of the next tick, players who matched the selector in the previous tick will have the `alreadyMatched` scoreboard tag. This means that we can select players who don't have the scoreboard tag (`tag=!alreadyMatched`, meaning they didn't match in the previous tick) but do *now* match the selector:

    execute as @a[x=73,y=10,z=3,distance=..1,tag=!alreadyMatched] run say I just entered the area!
    tag @a[tag=alreadyMatched] remove alreadyMatched
    tag @a[x=73,y=10,z=3,distance=..1] add alreadyMatched