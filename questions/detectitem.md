# Detect a specific item

## Java

Detecting an item by some property is a relatively easy task if you know your way around NBT. If you have control over how and when the item is given, [consider using a special item tag to detect it](/questions/customitemtag) (that post also contains useful information about how to nest the NBT, so read it before reading on).

Since not all NBT tags can be covered, this post will focus on the `id` and `Name` part of an item, assuming you're testing for a `minecraft:stick` with the name `Awesome Stick`.

To get the NBT for your specific usecase, use the `data get` command to see the complete NBT of the item.

    data get entity @s Inventory
    data get entity @e[type=item,limit=1,sort=nearest]
    data get entity @s SelectedItem
    data get block X Y Z Items

### In the inventory

We have to differentiate between different inventories: A players inventory is stored in the `Inventory` tag, a blocks (chest, barrel, hopper, etc) inventory is generally stored in the `Items` tag.

The item is _somewhere_ in the players inventory:

    @a[nbt={Inventory:[{id:"minecraft:stick",tag:{display:{Name:'{"text":"Awesome Stick"}'}}}]}]

The item is in a specific slot in the players inventory:

    @a[nbt={Inventory:[{Slot:0b,id:"minecraft:stick",tag:{display:{Name:'{"text":"Awesome Stick"}'}}}]}]

The item is _somewhere_ in a blocks inventory:

    execute if block ~ ~ ~ hopper{Items:[{id:"minecraft:stick",tag:{display:{Name:'{"text":"Awesome Stick"}'}}}]}

### In the selected slot

To get the selected slot, we can use the `SelectedItem` NBT:

    @a[nbt={SelectedItem:{id:"minecraft:stick",tag:{display:{Name:'{"text":"Awesome Stick"}'}}}}]

### On the ground

The items NBT is stored inside the `Item` NBT tag, so we can test for an item that is our stick like this:

    @e[type=item,nbt={Item:{id:"minecraft:stick",tag:{display:{Name:'{"text":"Awesome Stick"}'}}}}]

## Bedrock

In bedrock it is much more tricky to detect a specific item, and we are currently limited to detecting the item by name on the ground and by id in the players inventory.

### In the inventory

_Only detecting the item in a players inventory is currently possible with commands._

#### since 1.18.20

In the 1.18.20 beta they added the [`hasitem`](https://minecraft.fandom.com/wiki/Target_selectors#Selecting_targets_by_items) target selector, which allows you to check for specific amounts (as [ranges](/questions/range)) of specific items in specific amounts in entities inventories. Below are some examples, check the link above for more information.

A player with 5 or more apples in their inventory

    @a[hasitem={item:apple,quantity=5..}]

A player with an iron pickaxe in their mainhand

    @a[hasitem={item=iron_pickaxe,location=slot.weapon.mainhand}]

A player with a diamond in the first 10 slots of their enderchest

    @a[hasitem={item=diamond,location=slot.enderchest,slot=0..9}]

#### before 1.18.20

Using the `clear` command with a max count of 0 will return a successful result if the item is in the players inventory, without actually removing the item, which can be picked up by either a conditional commandblock or a comparator. **You can only do this for 1 player at a time though, as if you use `@a` you have no way of knowing which one of the players has the item in their inventory.**

    clear @p dirt 0 0
    # this second command is in a conditional commandblock
    execute @p ~ ~ ~ say I've got at least 1 normal dirt in my inventory!

_There is a way to detect the item the player is holding using behavior packs, animations and MoLang queries, which is too complicated to explain here. Look at the vanilla player animation controller file for an example._

If you want to know whether the player has more than one item in their inventory, you need to actually remove a single item through the clear command and then use the conditional command to count up a scoreboard score.  

If you then reach the desired amount, the player had the required amount of items in their inventory. If you don't reach it, they didn't have the required amount.  
You can then return the items one by one, one for each one you cleared.

### On the ground

Detecting a specific item on the ground involves using its name.

    @e[type=item,name="Dirt"]

**This method is language specific and won't work if your hosting player is not playing in english!** A way to circumvent that problem would be to either rename the item if possible, or to change every language file the game has to make the name of the item the same across all languages, which will require you to include a resourcepack.