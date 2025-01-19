# Select players with exactly X amount of items?

## Java

### Clear

Using the [`clear`](https://minecraft.wiki/Commands/clear) command with a `[<maxCount>]` of 0 will return the amount of items a player has, without actually clearing any of them.  
That means we can use [`execute store`](https://minecraft.wiki/w/Commands/execute#Store_subcommand) to save that number somewhere and then execute depending on that:

    # Setup
    scoreboard objectives add diamonds dummy

    # Commands
    execute store result score @s diamonds run clear @s diamond 0
    execute if score @s diamonds matches 5 run say I have exactly 5 diamonds in my inventory.
    execute if score @s diamonds matches 1..4 run say I have somewhere between 1 and 4 diamonds in my inventory.
    execute if score @s diamonds matches 10.. run say I have 10 or more diamonds in my inventory.
    execute if score @s diamonds matches ..20 run say I have 20 or less diamonds in my inventory.

### Execute if items

Since version 1.20.5, you can also use [`execute if items`](https://minecraft.wiki/w/Commands/execute#(if|unless)_items) to count the number of items.

The `if items` subcommand, when executed, returns the number of items that meet the specified conditions. For a quick example, running this command will show the count of all items in the player's inventory (except for armor and left hand slots):

    # In chat
    execute if items entity @s container.* *

You can find out more details on how to detect specific items here: [Detect a specific item](/wiki/questions/detectitem)

Below is an example for getting the amount of a custom item and executing some command depending on the result:

    # Example item
    give @s diamond[minecraft:custom_data={diamond:true},minecraft:item_name="'Custom Diamond'"]
    
    # Command blocks
    execute as @a store result score @s diamonds if items entity @s container.* *[custom_data~{diamond:true}]
    execute as @a[scores={diamonds=5}] run say I have exactly 5 coins.
    execute as @a[scores={diamonds=1..4}] run say I have somewhere between 1 and 4 coins.
    execute as @a[scores={diamonds=10..}] run say I have 10 or more coins.
    execute as @a[scores={diamonds=..20}] run say I have 20 or less coins.

Although you can use `/clear` on a player, `if items` can also count the number of items in a chest, shulker_box and other containers, can also count the number of items in a players ender_chest.

Here are some examples for this:

    # Counting items in chest or any container
    execute store result score #container diamonds if items block <pos> container.* *[custom_data~{diamond:true}]
    
    # Counting items in ender_chest
    execute as @a store result score @s diamonds if items entity @s enderchest.* *[custom_data~{diamond:true}]

## Bedrock

In the **1.18.20 beta** they added the [`hasitem`](https://minecraft.wiki/wiki/Target_selectors#Selecting_targets_by_items) target selector, which allows you to check for specific amounts (as [ranges](/wiki/questions/range)) of items in entities inventories. Below is an example (using 1.19.50 execute), check the link above for more information.

    execute as @a[hasitem={item=apple,quantity=5..}] run say I have 5 or more apples in my inventory
