# How to make a shop
This is a quick guide of how to make a shop where you can buy items with other items or costing a scoreboard value (money or coins).

## Item shop
This method consists of buying items with other items, in this example, you will buy 1 netherite ingot with 5 diamonds.

_Related:_ [Count how much of X item the player has](wiki/questions/amountitems)

> [!NOTE]
> You can add any command you want to run (for example a playsound) before the last command, but the selector must be the same as the command before.


### Java

#### Using commands
Both methods work the same, the just differ slightly in how they get the diamond count:
First we figure out how many diamonds the player has and store the value in a scoreboard. If the player has the required number of diamonds we give the netherite ingot and clear the diamonds from them.

Using command blocks we need to make sure we select the correct player every time, so with the first command we tag the relevant player to later limit our selector to that player.

To avoid the player getting unintentionally re-selected we remove the tag at the end.
Using functions we can rely on `@s` instead, assuming the function is executed `as` the relevant player.

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

> [!NOTE]
> The function `example:buy/netherite` must be run `as` the player

In 1.20.5 `execute if items` was added, which allows to count items in a diferent way.
To use this method, all command remains the same, as the previus example except the one for counting items that is this one:

    execute as @a[tag=buyer.netherite] store result score @s diamonds run clear @s diamond 0

That we can replace it for this one:

    execute as @p[tag=buyer.netherite] store result score @s diamonds if items entity @s container.* diamond
    
Or if you are using the example function, you need to replace this command

    execute store result score @s diamonds run clear @s diamond 0

With this command:

    execute store result score @s diamonds if items entity @s container.* diamond

> [!NOTE]
> The method that uses the `/clear` command will work in 1.20.5+ but it is recomended to use the one specific for that versions (using `execute if items`, that will not work below 1.20.5).

### Bedrock
In bedrock we have the `hasitem` argument, so it uses less commands than in Java.

#### With an NPC
This commands must be run in this order in an NPC

    tag @initiator[hasitem={item=diamond,quantity=5..}] add buy.netherite
    clear @initiator[tag=buy.netherite] diamond 5
    give @initiator[tag=buy.netherite] netherite_ingot 1
    tellraw @initiatior[tag=buy.netherite] {"rawtext":[{"text":"§2You bought a netherite ingot"}]}
    tellraw @initiator[tag=!buy.netherite] {"rawtext":[{"text":"§3You don't have 5 diamonds"}]}
    tag @initiator[tag=buy.netherite] remove buy.netherite

_Related: [How to setup a NPC?](wiki/questions/npc)_

#### Without an NPC
If you don't want to use an NPC can use this method, it is very similar to Java but it uses the `hasitem` argument instead.

    tag @p add buyer.netherite
    give @a[tag=buyer.netherite,hasitem={item=diamond,quantity=5..}] netherite_ingot
    tellraw @a[tag=buyer.netherite,hasitem={item=diamond,quantity=5..}] {"rawtext":[{"text":"§2You bought a netherite ingot"}]}
    tellraw @a[tag=buyer.netherite,hasitem={item=diamond,quantity=..5}] {"rawtext":[{"text":"§3You don't have 5 diamonds"}]}
    clear @a[tag=buyer.netherite,hasitem={item=diamond,quantity=5..}] diamond 5
    tag @a remove buyer.netherite
> [!NOTE]
> It is super important to clear the diamonds in the last step before removing the tag

### Add more than one items
In this gide we will use just one item, but you can have multiples but it will require a second tag, that must be added if the player has buth items.
Then when we clear the items we are going to clear them for the player with that tag.

In this example we will buy a gold block with 2 emeralds and 5 diamonds. If you are in Java you will need one scoreboard for each item, assuming you already store the items result in each one.

    # Example
    tag @p add buyer.example
    execute as @a[tag=buyer.example] run tag @s[scores={diamonds=5..,emeralds=2..}] add buy.example
    # run any tellraw to the player with the tag buy.example
    clear @a[tag=buy.example] diamond 5
    clear @a[tag=buy.example] emerald 2
    give @a[tag=buy.example] gold_block 1
And then we remove all the previus used tags:

    tag @a remove buy.example
    tag @a remove buyer.example

In bedrock use the `hasitem` argument instead of `scores`

## Score shop
This method uses a scoreboard as a currency (such as `coins` for this example) and you can buy items with that currency. In this example, you can buy a `diamond` with 10 `coins`.
You can adjust the items and the prices by editing the command item and quantity.

### Java and Bedrock
When the player wants to buy the item, run this commands in order. It can be achieved with a button, an impulse command block and a chain of command blocks behind as an example.
In this example the currency is a `dummy` scoreboard called `coins`.

#### With command blocks

> [!NOTE]
> This example uses Java syntax for the message that appears when the player buys the item

    /tag @p add buyer.diamond
    /execute as @p[tag=buyer.diamond] run tag @s[scores={coins=10..}] add buy.diamond
    /scoreboard players remove @a[tag=buy.diamond] coins 10
    /tellraw @a[tag=buyer_diamond,tag=!buy.diamond] "You need at least 10 coins"
    /tellraw @a[tag=buy.diamond] "You bought a diamond"
    /tag @a remove buy.diamond
    /tag @a remove buyer.diamond

#### In a function
This is more optimized compared to using command blocks as functions keep the context

    # function example:buy/diamond
    execute if entity @s[scores={coins=10..}] run function example:buy/diamond/success
    tellraw @s[scores={coins=..9}] {"text":"You don't have 10 coins","color":"red"}

    # function example:buy/diamond/success
    give @s diamond
    scoreboard players remove @s coins 10
    tellraw @s {"text":"You bought a diamond","color":"green"}

> [!NOTE]
> The function `example:buy/diamond` must be run `as` the player
