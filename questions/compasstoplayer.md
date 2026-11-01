# Make compass point towards player/location


## Java and Bedrock

Since the compass will always point to the world spawnpoint, your only option is to move the worldspawn to the player you want to point the compass towards. **This means it can only point to one player at a time and the pointing is global, meaning everyone sees the same compass pointing.**

**Java & Bedrock syntax:**

```mcfunction
execute at @p[<your target player>] run setworldspawn ~ ~ ~
```

| 📝 Note |
|---------|
|If a player dies, they will respawn at the world spawn (unless they have an individual spawn set), which this method is moving around|
|To avoid this issue, you need to detect when the players respawn to then teleport them to the intended respawn point. In Java edition you can use the scoreboard criteria `custom:time_since_last_death` but in bedrock you will need [a more complex way](https://wiki.bedrock.dev/commands/on-player-death)|

## 1.17+ (Java only)

In 1.17 we got [item modifiers](https://minecraft.wiki/wiki/Item_modifier), which allow us to modify an item live while it's in the player's inventory, no need to drop it on the floor anymore (if you know which slot it is in).

In this case you follow the process of the 1.16 section below, but instead of storing it into an item entity, you for example store it into the [command storage](https://minecraft.wiki/wiki/Commands/data#Storage) and then use the `copy_nbt` function in the item modifier to copy that data to the item or `set_components` with a macro for 1.20.5+.

In order to do this, we have an advancement of `location` trigger (for performance reasons, to not update the compass each tick) and with the condition of the player having in either weapon hand the custom compass.  
This rewards a function that will revoke the advancement and copy the data of the nearest player with the `target` tag (which is not the one holding the compass) to a storage for future use. Then from this storage we store in a separate one (the macro one) the values we care, `Pos[0]` (x), `Pos[2]` (y), `Dimension` and the hand we are holding the compass with (either `mainhand` or `offhand`).  
We store the data of the player in a separate storage first instead of storing only the data we care directly into the storage because of performance reasons, `data get storage` is better than `data get entity` so we try to minimize the use of the last one.  
Lastly, we run a macro function that is an item modifier that will modify the weapon hand we stored in a macro and will insert the correct values for `x`, `z` and dimension.

<details markdown="1">
  <summary style="color: #e67e22; font-weight: bold;">See datapack (1.20.5+)</summary>

```json
# advancement example:compasstoplayer/location
{
  "criteria": {
    "criteria": {
      "trigger": "minecraft:location",
      "conditions": {
        "player": {
          "slots": {
            "weapon.*": {
              "predicates": {
                "minecraft:custom_data": "{compasstoplayer:true}"
              }
            }
          }
        }
      }
    }
  },
  "rewards": {
    "function": "example:compasstoplayer/update_position"
  }
}
```
```mcfunction
# function example:compasstoplayer/get_otem
give @s compass[custom_data={compasstoplayer:true}]

## Use this function to get the compass tracker
## Note: add the tag "target" to the player that you want to point with the compass

# function example:compasstoplayer/update_position
advancement revoke @s only example:compasstoplayer/location
data modify storage example:data compasstoplayer set from entity @p[distance=0.1..,tag=target]
execute store result storage example:macro compasstoplayer.x int 1 run data get storage example:data player.Pos[0]
execute store result storage example:macro compasstoplayer.z int 1 run data get storage example:data player.Pos[2]
data modify storage example:macro compasstoplayer.dimension set from storage example:data player.Dimension
execute if items entity @s weapon.offhand *[custom_data~{compasstoplayer:true}] run data modify storage example:macro compasstoplayer.weapon set value "offhand"
execute if items entity @s weapon.offhand *[custom_data~{compasstoplayer:true}] run data modify storage example:macro compasstoplayer.weapon set value "mainhand"
function example:compasstoplayer/lodestone_update with storage example:macro lodestone

# function example:compasstoplayer/lodestone_update
$item modify entity @s weapon.$(weapon) {function:"minecraft:set_components",components:{"minecraft:lodestone_tracker":{target:{dimension:"$(dimension)",pos:[$(x),0,$(z)]},tracked:0b}}}
```
</details>

_Related: [Modify an item inside the player's inventory](/wiki/questions/modifyinventory)_

## 1.16+ (Java only)

With 1.16 a new functionality has been introduced into the game: a Lodestone Compass. It allows you to point an individual compass to a specific location without the need to change the worldspawn.

The give command for such a compass which doesn't need a lodestone to work looks like this:

```mcfunction
/give @s compass{LodestoneTracked:0b,LodestonePos:{X:-460,Y:64,Z:-701},LodestoneDimension:"minecraft:overworld"}
```

Now, using data merge, we can modify those numbers dynamically. For example, let's say you want to drop the compass to make it point to the second-nearest player, assuming the nearest player is you. The commands for something like that could look like this (using the [custom item tag](/wiki/questions/customitemtag) `playertracker:1b` to identify such a compass):

```mcfunction
# Command block / tick function
execute as @e[type=item,nbt={Item:{tag:{playertracker:1b}}}] at @s run function namespace:update_compass

# function update_compass
tag @p add closest
tag @p[tag=!closest] add target
data modify entity @s Item.tag.LodestonePos.X set from entity @p[tag=target] Pos[0]
data modify entity @s Item.tag.LodestonePos.Y set from entity @p[tag=target] Pos[1]
data modify entity @s Item.tag.LodestonePos.Z set from entity @p[tag=target] Pos[2]
tag @a remove closest
tag @a remove target
data modify entity @s PickupDelay set value 0s
```

(this assumes that the original given compass has the relevant tags (`LodestoneDimension`, `LodestoneTracked`, `LodestonePos`) already set to initial/correct values, as well as the aforementioned custom tag.)