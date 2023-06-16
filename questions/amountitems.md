# Select players with exactly X amount of items?

## Java

Using the [`clear`](https://minecraft.gamepedia.com/Commands/clear) command with a `[<maxCount>]` of 0 will return the amount of items a player has, without actually clearing any of them.  
That means we can use `execute store` to save that number somewhere and then execute depending on that:

    execute store result score @s diamonds run clear @s diamond 0
    execute if score @s diamonds matches 5 run say I have exactly 5 diamonds in my inventory.
    execute if score @s diamonds matches 1..4 run say I have somewhere between 1 and 4 diamonds in my inventory.
    execute if score @s diamonds matches 10.. run say I have 10 or more diamonds in my inventory.
    execute if score @s diamonds matches ..20 run say I have 20 or less diamonds in my inventory.

## Bedrock

In the **1.18.20 beta** they added the [`hasitem`](https://minecraft.fandom.com/wiki/Target_selectors#Selecting_targets_by_items) target selector, which allows you to check for specific amounts (as [ranges](/wiki/questionss/range)) of items in entities inventories. Below is an example, check the link above for more information.

    execute @a[hasitem={item:apple,quantity=5..}] ~~~ run say I have 5 or more apples in my inventory