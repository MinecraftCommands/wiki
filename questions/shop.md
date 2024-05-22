# How to make a shop
This is a quick guide of how to make a shop where you can buy items with other items or costing a scoreboard value (money or coins).

## Item shop
This method consists of buying items with other items, in this example, you will buy 1 netherite ingot with 5 diamonds.

> [!NOTE]
> You can add any command you want to run (for example a playsound) before the last command, but the selector must be the same as the command before, (`@a[tag=buy.netherite]` in this case)

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
    tag @p add buyer.netherite
    execute as @a[tag=buyer.netherite] store result score @s diamonds run clear @s diamond 0
    give @a[tag=buyer.netherite,scores={diamonds=5..}] netherite_ingot 1
    clear @a[tag=buyer.netherite,scores={diamonds=5..}] diamond 5
    tellraw @a[tag=buyer.netherite,scores={diamonds=5..}] {"text":"You bought a netherite ingot for 5 diamonds","color":"green"}
    tellraw @a[tag=buyer.netherite,scores={diamonds=..4}] {"text":"You don't have 5 diamonds","color":"dark_red"}
    tag @a remove buyer.netherite

Or if you prefer a function:

    # function example:load
    scoreboard objectives add diamonds dummy
    
    # function example:buy/netherite
    execute store result score @s diamonds run clear @s diamond 0
    execute if entity @s[scores={diamonds=5..}] run function example:buy/netherite/success
    execute unless entity @s[scores={diamonds=5..}] run tellraw @s {"text":"You don't have 5 diamonds","color":"dark_red"}

    # function example:buy/netheirte/success
    give @s netherite_ingot 1
    clear @s diamond 5
    tellraw @s {"text":"You bought a netherite ingot for 5 diamonds","color":"green"}
    ## you can add any command you want here

> [!NOTE]
> The function `example:buy/netherite` must be run `as` the player

#### 1.20.5 and above

    # Command blocks
    tag @p add buyer.netherite
    execute as @p[tag=buyer.netherite] store result score @s diamonds if items entity @s container.* diamond
    give @a[tag=buyer.netherite,scores={diamonds=5..}] netherite_ingot 1
    clear @a[tag=buyer.netherite,scores={diamonds=5..}] diamond 5
    tellraw @a[tag=buyer.netherite,scores={diamonds=5..}] {"text":"You bought a netherite ingot for 5 diamonds","color":"green"}
    tellraw @a[tag=buyer.netherite,scores={diamonds=..4}] {"text":"You don't have 5 diamonds","color":"dark_red"}
    tag @a remove buyer.netherite
    
Or if you prefer a function:

    # function example:load
    scoreboard objectives add diamonds dummy
    
    # function example:buy/netherite
    execute store result score @s diamonds if items entity @s container.* diamond
    execute if entity @s[scores={diamonds=5..}] run function example:buy/netherite/success
    execute unless entity @s[scores={diamonds=5..}] run tellraw @s ["",{"text":"You don't have 5 diamonds","color":"dark_red"}]
    
    # function example:buy/netherite/success
    give @s netherite_ingot 1
    clear @s diamond 5

> [!NOTE]
> The function `example:buy/netherite` must be run `as` the player

### Bedrock
In bedrock we have the `hasitem` argument, in this case this command would be from an NPC, if you don't want to use a NPC you can make them in a chain of command blocks and change `@initiator` to `@p[r=10]`.

    /execute as @initiator[hasitem={item=diamond,quantity=5..}] run tag @s add buy.netherite
    /clear @initiator[tag=buy.netherite] diamond 5
    /give @initiator[tag=buy.netherite] netherite_ingot 1
    /execute as @initiator[tag=buy.netherite] at @s run <any command>
    tellraw @initiatior[tag=buy.netherite] {"rawtext":[{"text":"§2You bought a netherite ingot"}]}
    /tag @initiator[tag=buy.netherite] remove buy.netherite
    /tellraw @initiator[hasitem={item=diamond,quantity=..4}] {"rawtext":[{"text":"§3You don't have 5 diamonds"}]}

## Score shop
This method uses a scoreboard as a currency (such as `coins` for this example) and you can buy items with that currency. In this example, you can buy a `diamond` with 10 `coins`.
You can adjust the items and the prices by editing the command item and quantity.

## Java and Bedrock
When the player wants to buy the item, run this commands in order. It can be archived with a button, an impulse command block and a chain of command blocks behind as an example.

### With command blocks

    /tag @p add buyer.diamond
    /execute as @p[tag=buyer.diamond] run tag @s[scores={coins=10..}] add buy_diamond
    /scoreboard players remove @a[tag=buy.diamond] coins 10
    /tellraw @a[tag=buyer_diamond,tag=!buy.diamond] "You need at least 10 coins"
    /tellraw @a[tag=buy.diamond] "You bought a diamond"
    /tag @a remove buy.diamond
    /tag @a remove buyer.diamond

### In a function
This is more optimized compared to using command blocks as functions keep the selector

    # function example:buy/diamond
    execute if entity @s[scores={coins=10..}] run function example:buy/diamond/success
    tellraw @s[scores={coins=..9}] {"text":"You don't have 10 coins","color":"red"}

    # function example:buy/diamond/success
    give @s diamond
    scoreboard players remove @s coins 10
    tellraw @s {"text":"You bought a diamond","color":"green"}

> [!NOTE]
> The function `example:buy/diamond` must be run `as` the player
