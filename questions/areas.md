# Do something if a player is in certain areas

> [!NOTE]
> Java Syntax, but this can be applied to Bedrock just as well by changing the selector arguments to Bedrock Syntax. Instead of `distance=..X`, use `r=X` and instead of `distance=X..Y`, use `r=Y,rm=X`.

This mostly comes up as a question to change the gamemode in a certain area (e.g. spawn, safe zones, etc.), so we will focus on that, but this can be applied to any use case. For questions to do something once a player enters a single area, [look here](/wiki/questions/runonce).

The naive approach would be to put everyone in a radius X around the worldspawn (e.g. at 0 0 0) to adventure mode and everyone outside of it to survival mode.

    gamemode adventure @a[x=0,y=0,z=0,distance=..X]
    gamemode survival @a[x=0,y=0,z=0,distance=X..]

This method quickly falls apart if you have a non-spherical or non-box shaped area or you want this to apply to more than one area, because a player will always be outside of the other areas, even if they are inside one, so will always end up in survival.

## Hardcoded locations

The best way to make sure players are in one of multiple areas without overwriting each other, is to use a tag: So instead of applying the desired effect to each area individually, you tag all players that are in one of the areas and apply the effect once to all of them (or everyone else). But this method requires a separate command block for each location. For a large number of locations, use the anchor entity method.

    # Command blocks / tick function
    tag @a remove inArea
    tag @a[x=0,y=0,z=0,distance=..X] add inArea
    tag @a[x=100,y=64,z=100,dx=70,dy=16,dz=28] add in Area
    ....
    gamemode adventure @a[tag=inArea]
    gamemode survival @a[tag=!inArea]

To prevent chatspam for the players and unnecessary gamemode changes, we suggest using the gamemode selector argument on the last two commands:

    gamemode adventure @a[tag=inArea,gamemode=!adventure]
    gamemode survival @a[tag=!inArea,gamemode=!survival]

If for some reason you want to keep commandBlockOutput on and don't want your output to be spammed by this system, check out [this post](https://www.reddit.com/r/MinecraftCommands/comments/mw11xm/do_something_to_players_in_multiple_specific) by [u/Afanofall23](https://www.reddit.com/u/Afanofall23).

## Anchor entities

If you need to check if the player is in one of several spherical areas, for example to switch gamemode to adventure, then you can use some kind of entity as an anchor to check if the player is nearby. For versions before 1.17 you can use armor_stand or area_effect_cloud, but since version 1.17 it is strongly recommended to use an [marker entity](https://minecraft.wiki/w/Marker) to mark a place.

    # Summon marker
    summon marker <pos> {Tags:["adventure_area"]}
    
    # Command blocks / tick function
    execute as @a[gamemode=survival] at @s if entity @e[type=marker,tag=adventure_area,distance=..X,limit=1] run gamemode adventure
    execute as @a[gamemode=adventure] at @s unless entity @e[type=marker,tag=adventure_area,distance=..X,limit=1] run gamemode survival
    
`X - distance at which players should be switched into adventure mode.`

To make placing markers more convenient, you can give a spawn_egg containing a marker with the tag:

    # 1.17 - 1.20.4
    give @s bat_spawn_egg{EntityTag:{id:"minecraft:marker",Tags:["adventure_area"]},display:{Name:'{"text":"Adventure Area Marker","italic":false}'}}
    
    # 1.20.5+
    give @s minecraft:bat_spawn_egg[entity_data={id:"minecraft:marker",Tags:["adventure_area"]},item_name='"Adventure Area Marker"']

## Block layer

If you need to execute a command when a player enters a very randomly shaped area, then you can place under the map, for example, at a height of Y=-63, a layer of some block that you will check under the player.

For example, you want to create an area on your map where the player will be detected if the player is not sneaking. This example will check the red_concrete block at Y=-63 for this area:

    # Command block / tick function (1.20.5+)
    execute as @a at @s if predicate {condition:"entity_properties",entity:"this",predicate:{flags:{is_sneaking:false}}} if block ~ -63 ~ minecraft:red_concrete run say You have been found!

## Predicates

If you need to check multiple cubic, spherical or cylindrical areas, you can also use [predicates](https://minecraft.wiki/w/Predicate) in datapack or command blocks (1.20.5+).

In a predicate, you can use the `minecraft:alternative` (1.14-1.19.4) or `minecraft:any_of` (1.20+) condition to check multiple areas in one predicate using the `minecraft:location_check` condition.

Below is an example of a predicate for checking three cubic areas:

    {
      "condition": "minecraft:any_of",
      "terms": [
        {
          "condition": "minecraft:location_check",
          "predicate": {
            "position": {
              "x": {
                "min": 10,
                "max": 20
              },
              "y": {
                "min": 64,
                "max": 70
              },
              "z": {
                "min": 30,
                "max": 40
              }
            }
          }
        },
        {
          "condition": "minecraft:location_check",
          "predicate": {
            "position": {
              "x": {
                "min": 60,
                "max": 85
              },
              "y": {
                "min": -20,
                "max": 10
              },
              "z": {
                "min": 10,
                "max": 80
              }
            }
          }
        },
        {
          "condition": "minecraft:location_check",
          "predicate": {
            "position": {
              "x": {
                "min": -80,
                "max": -20
              },
              "y": {
                "min": 125,
                "max": 155
              },
              "z": {
                "min": 55,
                "max": 78
              }
            }
          }
        }
      ]
    }
