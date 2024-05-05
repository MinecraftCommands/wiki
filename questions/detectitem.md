# Detect a specific item

## Java

Detecting an item by some property is a relatively easy task if you know your way around NBT. If you have control over how and when the item is given, [consider using a special item tag to detect it](/wiki/questions/customitemtag) (that post also contains useful information about how to nest the NBT, so read it before reading on).

Since not all NBT tags can be covered, this post will focus on the `id` and `Name` part of an item, assuming you're testing for a `minecraft:stick` with the name `Awesome Stick`.

To get the NBT for your specific usecase, use the `data get` command to see the complete NBT of the item.

    data get entity @s Inventory
    data get entity @e[type=item,limit=1,sort=nearest]
    data get entity @s SelectedItem
    data get block X Y Z Items

Let's give a simple example of an item with a custom name and [custom tag](/questions/customitemtag.md):

    # 1.13 - 1.20.4
    give @s minecraft:stick{awesome_stick:true,display:{Name:'{"text":"Awesome Stick"}'}}

    # 1.20.5+
    give @s minecraft:stick[minecraft:custom_data={awesome_stick:true},minecraft:custom_name='"Awesome Stick"']

**Important!** When you create any custom item that the player should not be able to obtain by renaming it at an anvil, always add a [custom tag](/questions/customitemtag.md) to your item and when check only this tag, but not the item name, because checking the item name can cause problems with proper formatting and makes the command longer. All examples of check items below will be duplicated with checking the item name and checking the custom tag, if applicable.

While holding this item in your hand you can get the item data using /data get in chat:

    data get entity @s SelectedItem

Due to changes in different versions, you may receive different results in the chat:

    # 1.13 - 1.20.1
    {id:"minecraft:stick",Count:1b,tag:{awesome_stick:true,display:{Name:'{"text":"Awesome Stick"}'}}}

    # 1.20.2 - 1.20.4
    {id:"minecraft:stick",Count:1b,tag:{awesome_stick:true,display:{Name:'"Awesome Stick"'}}}

    # 1.20.5+
    {id:"minecraft:stick",count:1,components:{"minecraft:custom_data":{awesome_stick:true},"minecraft:custom_name":'"Awesome Stick"'}}

As of version 1.20.2, any [JSON text](https://minecraft.wiki/w/Raw_JSON_text_format) will now be represented as plain text, if applicable. So if you give the player an item named `'{"text":"Some Text"}'` it will automatically be converted to `'"Some Text"'` unless you change the color, italic, or any other formatting. This applies not only to items, but also to books, signs, and any other places where JSON text is used.

**Important!** When checking NBT data, you can only check the data you need, so you don't have to check the item ID or Count tag if it's not important to you, but you can't use the shorthand syntax that is available with the /give command.
So, you can give yourself an item with this command:

    give @s apple[custom_data={some_tag:1}]

But you will **NOT** be able to check this item with this NBT receipt:

    {SelectedItem:{id:apple,components:{custom_data:{some_tag:1}}}}

The correct check in the `SelectedItem` slot would look something like this:

    {SelectedItem:{id:"minecraft:apple",components:{"minecraft:custom_data":{some_tag:1}}}}

*If you are not sure what data structure an item has, use `/data get` to get the correct data.*

## 1.20.5 and above
 
### Target selector

In 1.20.5 you can check an item using the NBT data check in the [target selector](https://minecraft.wiki/w/Target_selectors#Selecting_targets_by_nbt), however now can use [`execute if items`](https://minecraft.wiki/w/Commands/execute#(if|unless)_items) to flexibly detect items and can now use the [predicate](https://minecraft.wiki/w/Predicate) not only for equipment, but also for any slot and now even without using a datapack.

    # Any slot
    @a[nbt={Inventory:[{id:"minecraft:stick",components:{"minecraft:custom_data":{awesome_stick:true}}}]}]
    @a[nbt={Inventory:[{id:"minecraft:stick",components:{"minecraft:custom_name":'"Awesome Stick"'}}]}]
    
    # Specific slot
    @a[nbt={Inventory:[{Slot:0b,id:"minecraft:stick",components:{"minecraft:custom_data":{awesome_stick:true}}}]}]
    @a[nbt={Inventory:[{Slot:0b,id:"minecraft:stick",components:{"minecraft:custom_name":'"Awesome Stick"'}}]}]
    
    # Mainhand
    @a[nbt={SelectedItem:{id:"minecraft:stick",components:{"minecraft:custom_data":{awesome_stick:true}}}}]
    @a[nbt={SelectedItem:{id:"minecraft:stick",components:{"minecraft:custom_name":'"Awesome Stick"'}}}]

**Note:** The component `“minecraft:custom_data”` is escaped with parentheses because it contains the special character colon. And although you can omit `minecraft:` in /give and other commands, when checking NBT data in the target selector you should always specify the full format, which also includes the [namespace](https://minecraft.wiki/w/Resource_location#Namespaces).

### execute if items

The syntax looks like this [\[wiki\]](https://minecraft.wiki/w/Commands/execute#(if|unless)_items):

    if/unless items block <pos> <slots> <item_predicate>
    if/unless items entity <entities> <slots> <item_predicate>

`<slots>` - a specific [slot](https://minecraft.wiki/w/Slot) (`hotbar.3`) or a range of slots (`hotbar.*`). *Ranges as in `distance=1..5` are not allowed.*

`<item_predicate>` - specific item (`minecraft:yellow_wool`), item tag (`#minecraft:banners`) or any item (`*`). Checking a components or item sub-predicate is also supported.

An example for checking an item in almost any player slot:

    execute as @a if items entity @s container.* minecraft:stick[minecraft:custom_data~{awesome_stick:true}]

This will not include the offhand slot, armor slots and ender_chest slots, so it will require an additional command to check these slots or use a predicate in the datapack.

Can also check multiple items by checking the item tag, for example, if the player is holding any banner in his hand:

    execute as @a if items entity @s weapon.mainhand #minecraft:banners

Or can omit the item id check and check only the components. There are two modes for checking components - exact compliance with the specified condition (`=`) or checking the item as a sub-predicate (`~`).

The component check (`=`) checks the exact match of the component specified in the check and any difference from the specified one fails the check, so it is recommended to use the exact check only if the item sub-predicate (`~`) check is not available for this component. So, for example, if for some reason you want to check the `minecraft:can_break` component, then only the predicate check is available here, so you cannot find any item that, for example, can break a stone, but you always need to specify the entire component.

*Therefore, in this article, all `custom_data` component checks are used as item sub-predicate (`~`).*

In this example, any item in the hotbar with the unbreaking enchantment is detected, but if the item has any other enchantment, or enchantment level, then the check will fail for that item:

    execute as @a if items entity @s hotbar.* *[minecraft:enchantments={levels:{"minecraft:unbreaking":1}}]

But if you want this to work if the item has a different enchantment, or enchantment level, you need to use the item sub-predicate (~) for this. Here the syntax is the same as checking item data in a predicate:

    execute as @a if items entity @s hotbar.* *[minecraft:enchantments~[{"enchantment":"minecraft:unbreaking"}]]
    execute as @a if items entity @s hotbar.* *[minecraft:enchantments~[{enchantment:"minecraft:unbreaking",levels:{min:1,max:3}}]]

Item sub-predicate also allows you to detect an item with damage not with a specific value, but with a range or remaining durability:

    execute as @a if items entity @s weapon *[minecraft:damage~{damage:{min:5}}]
    execute as @a if items entity @s weapon *[minecraft:damage~{durability:{max:10}}]

Here in the first example it will detect an item that has at least 5 damage. The second example detects an item that has durability for no more than 10 uses.

But in addition to AND checks, you can check OR conditions.

This is an example of checking an item that has no more than 5 damage, OR more than 40 damage.

    execute as @a if items entity @s hotbar.* *[minecraft:damage~{damage:{max:5}}|minecraft:damage~{damage:{min:40}}]

Using `execute if items` you can check for an item not only in the player's inventory, but also in any slot for entity, [block entity](https://minecraft.wiki/w/Block_entity), item_frame, any projectile, or item on the ground.

You can check any slot block entity (chest, furnace, shulker_box, etc.) using container.<num> for a specific slot or container.* for any slot:

    execute if items block ~ ~ ~ container.* *[minecraft:custom_data~{awesome_stick:true}]

Another good example is checking the number of items:

    execute if items block ~ ~ ~ container.* *[minecraft:count~{min:8}]

This will check every slot in a chest or any container and find all item stacks with 8 or more items. But the result will not be the count of met slots in the condition, but the count of all items in all these slots.

To check for an item inside an item_frame, projectile or an item on the ground, use `container.0` or `contents` slot:

    execute as @e[type=item] if items entity @s contents minecraft:stick[minecraft:custom_data~{awesome_stick:true}]

### Predicate

When using predicates in a datapack, you can now check not only equipment slots, but any slot. Here, just like when using if items, you can check for an exact match of components or use item sub-predicate for more flexible item detection. Also, "items" now accepts one item, one item tag (separate "tag" has been removed), or a list of items.

This is an example of updating a predicate to detect an item with a custom tag:

```
{
  "condition": "minecraft:entity_properties",
  "entity": "this",
  "predicate": {
    "slots": {
      "weapon.mainhand": {
        "items": "minecraft:stick",
        "count": {
          "min": 1
        },
        "predicates": {
          "minecraft:custom_data": {"awesome_stick": true}
        }
      }
    }
  }
}
```

**Note:** If in the predicate you do not check the item ID, but only the NBT data of the item in the mainhand, then always also check the count item in this slot because of a bug [\[MC-229882\]](https://bugs.mojang.com/browse/MC-229882).

But in addition to creating predicates in a datapack, you can now use any predicates in command blocks without using a datapack.
Below is an example of checking the weather_check predicate, just as an example, you can use any predicate in this way:
```
execute if predicate {condition: 'minecraft:weather_check', raining: true} run say Is raining
```

## 1.20.4 and below

### In the inventory

We have to differentiate between different inventories: A players inventory is stored in the `Inventory` tag, a blocks (chest, barrel, hopper, etc) inventory is generally stored in the `Items` tag.

The item is _somewhere_ in the players inventory:

    @a[nbt={Inventory:[{id:"minecraft:stick",tag:{display:{Name:'{"text":"Awesome Stick"}'}}}]}]
    @a[nbt={Inventory:[{id:"minecraft:stick",tag:{awesome_stick:true}}]}]

The item is in a specific slot in the players inventory:

    @a[nbt={Inventory:[{Slot:0b,id:"minecraft:stick",tag:{display:{Name:'{"text":"Awesome Stick"}'}}}]}]
    @a[nbt={Inventory:[{Slot:0b,id:"minecraft:stick",tag:{awesome_stick:true}}]}]

The item is _somewhere_ in a blocks inventory:

    execute if block ~ ~ ~ hopper{Items:[{id:"minecraft:stick",tag:{display:{Name:'{"text":"Awesome Stick"}'}}}]}
    execute if block ~ ~ ~ hopper{Items:[{id:"minecraft:stick",tag:{awesome_stick:true}}]}

### In the selected slot

To get the selected slot, we can use the `SelectedItem` NBT:

    @a[nbt={SelectedItem:{id:"minecraft:stick",tag:{display:{Name:'{"text":"Awesome Stick"}'}}}}]
    @a[nbt={SelectedItem:{id:"minecraft:stick",tag:{awesome_stick:true}}}]

### On the ground

The items NBT is stored inside the `Item` NBT tag, so we can test for an item that is our stick like this:

    @e[type=item,nbt={Item:{id:"minecraft:stick",tag:{display:{Name:'{"text":"Awesome Stick"}'}}}}]
    @e[type=item,nbt={Item:{id:"minecraft:stick",tag:{awesome_stick:true}}}]

### Use predicate

When using a datapack, you can check items in equipment slots. To do this, create a file `data/<namespace>/predicates/<predicate_name>.json` in your datapack. For example, the predicate `data/example/predicates/has/awesome_stick.json` would have the resourcename `example:has/awesome_stick`. And you can check the predicate in any target selector, `execute if predicate <predicate>`, as well as in loot and advancements tables.
Here is an example of a predicate and its use in a target selector:
```
execute as @a[predicate=example:has/awesome_stick] run say Example Command. 

# predicate example:has/awesome_stick
{
  "condition": "minecraft:entity_properties",
  "entity": "this",
  "predicate": {
    "equipment": {
      "mainhand": {
        "items": [
          "minecraft:stick"
        ],
        "count": {
          "min": 1
        },
        "nbt": "{awesome_stick:true}"
      }
    }
  }
}
```

**Note:** If in the predicate you do not check the item ID, but only the NBT data of the item in the mainhand, then always also check the count item in this slot because of a bug [\[MC-229882\]](https://bugs.mojang.com/browse/MC-229882).

## Bedrock

In bedrock it is much more tricky to detect a specific item, and we are currently limited to detecting the item by name on the ground and by id in the players inventory.

### In the inventory

_Only detecting the item in a players inventory is currently possible with commands._

#### since 1.18.20

In the 1.18.20 beta they added the [`hasitem`](https://minecraft.wiki/wiki/Target_selectors#Selecting_targets_by_items) target selector, which allows you to check for specific amounts (as [ranges](/questions/range.md)) of specific items in specific amounts in entities inventories. Below are some examples, check the link above for more information.

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
