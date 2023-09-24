# Modify an item inside a players inventory

_Java only, as NBT data is inaccessible in bedrock this article is irrelevant for BE._

## 1.17 and above

In snapshot [20w46a](https://www.minecraft.net/article/minecraft-snapshot-20w46a) the `/replaceitem` command was replaced by the `item` command, which, while keeping the replacing functionality from the now obsolete command, also allows for modifiers to be applied to (replaced) items and items to be copied from other sources. This is currently only in the snapshots and subject to change. However if it stays around for the full release, it would drastically improve on the previous methods of doing this.

Further information: https://minecraft.wiki/Commands/item

## 1.16 and below

_As of 1.15, MC-123307, which allowed execute store and data modify commands to edit a player's inventory or ender chest items, has been fixed. The offical replacement was introduced in early 1.17 snapshots. However, there is still a trick dating back to 1.14 that makes use of the loot command with a modified shulker box loot table._

### The method:

In vanilla without data packs, shulker boxes drop themselves with their contents inside. Since 1.14 loot table changes, it is however possible to create a loot table that directly drops the box's contents to the ground instead. The loot replace command can then be used to simulate the destruction of said loot table and have it drop its contents directly to a player's inventory.  

Of course, that would change normal behavior of loot tables. The actual loot table consists of two entries, one which keeps the normal shulker box drop, and one that drops its contents when mined with a specific item: minecraft:air{drop_contents: 1b}. Because air cannot have NBT data, a player cannot notice any difference, whatever they are holding. The only case where this is useful is in the loot command.  

A shulker box is placed somewhere out of sight of the player. Sometimes far away, usually in a forceloaded chunk, like some libraries (Phi, AESTD, Lantern) do. Alternatively you can place it e.g. at ~ 0 ~ and replace it back with bedrock at the end of the thing. _It is desireable to not constantly setblock and replace a block though, so if you can place it somewhere permanently, do that._ An item is placed inside the shulker box, and is injected to the player's inventory.

### The loot table

This yellow shulker box loot table was designed by Luetzie for Lantern, and is also used by Phi and AESTD. If you use the exact same loot table, you will have no conflicts with these data packs. **It has been established as the standard loot_table for this endeavour by this commands community, and it is highly suggested you use this one!**

You can download it from here: https://lanternmc.com/yellow_shulker_box.json

### The commands:

Initialisation 

    # A shulker box is placed somewhere.
    setblock <pos> minecraft:yellow_shulker_box

Editing an item 

    # First, we copy the item to a storage.
    data modify storage lilith:example Item set from entity <player> SelectedItem
    
    # We can now modify what we want about the item here. Its 'Slot' is set to 0.
    data modify storage lilith:example Item merge value {tag: {Enchantments: [{id: "minecraft:knockback", lvl: 1s}]}}
    data modify storage lilith:example Item.Slot set value 0b
    
    # Third, we copy the storage item to the shulker box. Because 'Slot' is 0, the item will be in the first slot.
    data modify block <pos> Items append from storage lilith:example Item

    # Then, we move the shulker box item back to the player's inventory.
    loot replace entity <player> weapon.mainhand 1 mine <pos> minecraft:air{drop_contents: 1b}

_Credit to Lilith on our Discord for writing this explanation and posting it into the #resources channel on there._