# Make compass point towards player

Since the compass will always point to the worldspawnpoint, your only option is to move the worldspawn to the player you want to point the compass towards. **This means it can only point to one player at a time and the pointing is global, meaning everyone sees the same compass pointing.**

**Java & Bedrock syntax:**

    execute at @p[<your target player>] run setworldspawn ~ ~ ~

## 1.16+ (Java only)

With 1.16 a new functionality has been introduced into the game: a Lodestone Compass. It allows you to point an individual compass to a specific location without the need to change the worldspawn.

The give command for such a compass which doesn't need a lodestone to work looks like this:

    /give @s compass{LodestoneTracked:0b,LodestonePos:{X:-460,Y:64,Z:-701},LodestoneDimension:"minecraft:overworld"}

Now, using data merge we can modify those numbers dynamically. For example, lets say you want to drop the compass to make it point to the second-nearest player, assuming the nearest player is you. The commands for something like that could look like this (using the [custom item tag](/wiki/questions/customitemtag) `playertracker:1b` to identify such a compass):

tick.mcfunction

    execute as @e[type=item,nbt={Item:{tag:{playertracker:1b}}}] at @s run function namespace:update_compass

update_compass.mcfunction

    tag @p add closest
    tag @p[tag=!closest] add target
    data modify entity @s Item.tag.LodestonePos.X set from entity @p[tag=target] Pos[0]
    data modify entity @s Item.tag.LodestonePos.Y set from entity @p[tag=target] Pos[1]
    data modify entity @s Item.tag.LodestonePos.Z set from entity @p[tag=target] Pos[2]
    tag @a remove closest
    tag @a remove target
    data modify entity @s PickupDelay set value 0s

(this assumes that the original given compass has the relevant tags (`LodestoneDimension`, `LodestoneTracked`, `LodestonePos`) already set to initial/correct values, as well as the aforementioned custom tag.)

## 1.17+ (Java only)

In 1.17 we got [item modifiers](https://minecraft.wiki/wiki/Item_modifier), which allow us to modify an item live while it's in the players inventory, no need to drop it on the floor anymore (if you know which slot it is in).

In this case you follow the process of the 1.16 section above, but instead of storing it into an item entity you for example store it into the [command storage](https://minecraft.wiki/wiki/Commands/data#Storage) and then use the `copy_nbt` function in the item modifier to copy that data to the item.

_Related: [Modify an item inside the players inventory](/wiki/questions/modifyinventory)_