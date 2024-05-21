# How to make a shop
This is a quick guide of how to make a shop where you can buy items with other items or costing a scoreboard value (money or coins).

## Item shop
This method consists of buying items with other items, in this example, you will buy 1 netherite ingot with 5 diamonds.

> [!NOTE]
> You can add any command you wan't to run (for example a playsound) before the last command, but the selector must be the same as the command before, (`@a[tag=buy_netherite]` in this case)

### Java
There are 2 methods, one for pre-1.20.5 and the other for 1.20.5 and above, this is due to the new `execute if items` subcommand.
> [!NOTE]
> The method for pre-1.20.5 will still work in 1.20.5 and above.

Both methods will require this setup:

    # In chat / Load function
    scoreboard objectives add diamonds dummy

Both methods works the same:
First we are going to know how many diamonds the player has storing the value in a scoreboard and if the player has that that number of diamonds we will add a tag (first and second command) or we will run a function instead if using a datapack.
After that we are going to give the netherite ingot to the player with the tag and we will clear the diamonds from it.
To avoid the player from a loop we will remove the tag as we dont need it more.

#### Pre-1.20.5

    # Command blocks
    execute as @p store result score @s diamonds run clear @s diamond 0
    execute as @p run tag @s[scores={diamonds=5..}] add buy_netherite
    give @a[tag=buy_netherite] netherite_ingot 1
    clear @a[tag=buy_netherite] diamond 5
    tag @a remove buy_netherite

Or if you prefer a function:

    # function example:load
    scoreboard objectives add diamonds dummy
    
    # function example:buy/netherite
    execute store result score @s diamonds run clear @s diamond 0
    execute if entity @s[scores={diamonds=5..}] run function example:buy/netherite/success

    # function example:buy/netheirte/success
    give @s netherite_ingot 1
    clear @s diamond 5
    ## you can add any command you want here


#### 1.20.5 and above

    # Command blocks
    execute as @p store result score @s diamonds if items entity @s container.* diamond
    execute as @p at @s[scores={diamonds=5..}] run tag @s add buy_netherite
    give @a[tag=buy_item_1] netherite_ingot 1
    clear @a[tag=buy_netherite] diamond 5
    tag @a remove buy_netherite
    
Or if you prefer a function:

    # function example:load
    scoreboard objectives add diamonds dummy
    
    # function example:buy/netherite
    execute store result score @s diamonds if items entity @s container.* diamond
    
    # function example:buy/netherite/success
    give @s netherite_ingot 1
    clear @s diamond 5



### Bedrock
In bedrock we have the `hasitem` argument, in this case this command would be from an NPC, if you don't want to use a NPC you can make them in a chain of command blocks and change `@initiator` to `@p[r=10]`.

    /execute as @initiator[hasitem={item=diamond, quantity=5..}] run tag @s add buy_netherite
    /clear @initiator[tag=buy_netherite] diamond 5
    /give @initiator[tag=buy_netherite] netherite_ingot 1
    /execute as @initiator[tag=buy_netherite] at @s run <any command>
    /tag @initiator[tag=buy_netherite] remove buy_netherite

## Score shop
This method uses a scoreboard as a currency (such as `coins` for this example) and you can buy items with that currency. In this example, you can buy a `diamond` with 10 `coins`.
You can adjust the items and the prices by editing the command item and quantity.

## Java and Bedrock
When the player wants to buy the item, run this commands in order. It can be archived with a button, an impulse command block and a chain of command blocks behind as an example.

### With command blocks

    /execute as @p run tag @s[scores={coins=10..}] add buy_diamond
    /execute as @a[tag=buy_diamond] run <any command>
    /scoreboard players remove @a[tag=buy_diamond] coins 10
    /tag @a remove buy_diamond

### In a function
This is more optimized compared to using command blocks as functions keep the selector

    # function example:buy/diamond
    execute as @p at @s[scores={coins=10..}] run function example:buy/diamond/succes

    # function example:buy/diamond/succes
    give @s diamond
    scoreboard players remove @s coins 10
    ## you can add any command you want here
