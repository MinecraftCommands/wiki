# Modify an item inside a players inventory

_Java only, as NBT data is inaccessible in bedrock this article is irrelevant for BE._

## 1.20.5 and above

In this version, the functionality of [item_modifier](https://minecraft.wiki/w/Item_modifier) has been expanded with the ability to filter selected items for modification and the ability to change item ID. And the /item command now has the ability to use item_modifier / [predicates](https://minecraft.wiki/w/Predicate) directly in the game without using a datapack. Values have same structure as matching JSON files, though they are encoded as SNBT.

Quick example:

    execute if predicate {condition:weather_check, raining:true}

Here's a simple example of how to add a [food component](https://minecraft.wiki/w/Data_component_format#food) to any item in a player's hand (if the item is not food) without replacing the item completely:

    execute as @a if items entity @s weapon *[!minecraft:food] run item modify entity @s weapon {function:"minecraft:set_components", components: {"minecraft:food": {nutrition:1, saturation:2, can_always_eat:true, eat_seconds:3.2}}}

You can also change the item ID without changing any components:

    execute as @a run item modify entity @s weapon {function:"minecraft:filtered", item_filter: {items:"minecraft:iron_sword"}, modifier: {function:"minecraft:set_item", item:"minecraft:golden_sword"}}

This example uses the [`minecraft:filtered`](https://minecraft.wiki/w/Item_modifier#:~:text=or%20killer_player.-,filtered,-%E2%80%94Applies%20another%20function) loot function to check that the selected item is `minecraft:iron_sword` and not just any item and then uses the [`minecraft:set_item`](https://minecraft.wiki/w/Item_modifier#:~:text=is%20selected%20randomly.-,set_item,-%E2%80%94Replaces%20item%20type) function to replace iron_sword with golden_sword. In this case, any custom_data, damage, enchantments and other components that this item contains will not be changed, with the exception of inaccessible data, for example, if the original item had a [max_stack_size](https://minecraft.wiki/w/Data_component_format#max_stack_size) component greater than 1 and after modification you change to an item with a [max_damage](https://minecraft.wiki/w/Data_component_format#max_damage) component, then max_stack_size will be changed by 1.

## 1.17 and above

In snapshot [20w46a](https://www.minecraft.net/article/minecraft-snapshot-20w46a) the [`/replaceitem`](https://minecraft.wiki/w/Commands/replaceitem) command was replaced by the [`/item`](https://minecraft.wiki/Commands/item) command, which, while keeping the replacing functionality from the now obsolete command, also allows for modifiers to be applied to (replaced) items and items to be copied from other sources.

### item replace

With this command you can copy an item to a specified entity/block slot from an entity/block slot. To do this, you also need to use any entity/block that will be used as a buffer slot in which you can change the item data.
Here's a quick example of adding an enchantment to an item in a player's hand:

```
# Setup
setblock chest <pos>

# Commands
## Copying an item from a player to a chest
item replace block <pos> container.0 from entity <player> weapon.mainhand

## Applying item changes
data modify block <pos> Items[0] merge value {tag: {Enchantments: [{id: "minecraft:knockback", lvl: 1s}]}}

## Returning an item to the player
item replace entity <player> weapon.mainhand from block <pos> container.0
```

_Note: Without using a datapack, you can only do this for 1 item per tick._

### item modify

By using a datapack you can avoid copying the item to another location, and simply apply the item modifier to the specified player slot. To do this, you need to create an [item modifier](https://minecraft.wiki/w/Item_modifier) in the datapack with what you want to change. Since version 1.20.5 this also supports changing item ID.
Below is a small example of using the item modifier:

```
# Command
item modify entity <player> weapon.mainhand example:add_knockback

# item_modifier example:add_knockback (data/example/item_modifiers/add_knockback.json)
{
  "function": "minecraft:set_enchantments",
  "enchantments": {
    "knockback": 1
  },
  "add": true
}
```

## 1.16 and below

_As of 1.15, [MC-123307](https://bugs.mojang.com/browse/MC-123307), which allowed execute store and data modify commands to edit a player's inventory or ender chest items, has been fixed. The offical replacement was introduced in early 1.17 snapshots. However, there is still a trick dating back to 1.14 that makes use of the loot command with a modified shulker box loot table._

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

<details>
  <summary style="color: #e67e22; font-weight: bold;">See example</summary>

```py
# First, we copy the item to a storage.
data modify storage lilith:example Item set from entity <player> SelectedItem

# We can now modify what we want about the item here. Its 'Slot' is set to 0.
data modify storage lilith:example Item merge value {tag: {Enchantments: [{id: "minecraft:knockback", lvl: 1s}]}}
data modify storage lilith:example Item.Slot set value 0b

# Third, we copy the storage item to the shulker box. Because 'Slot' is 0, the item will be in the first slot.
data modify block <pos> Items append from storage lilith:example Item

# Then, we move the shulker box item back to the player's inventory.
loot replace entity <player> weapon.mainhand 1 mine <pos> minecraft:air{drop_contents: 1b}
```
</details>

_Credit to Lilith on our Discord for writing this explanation and posting it into the #resources channel on there._
