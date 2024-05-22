# Link an entity to another entity / to a player

_Also known as a **scoreboard ID system**_.

Sometimes there is a need to link two entites together in a logical fashion. In Minecraft, we can achieve this by giving both entities the same scoreboard score. In this article we'll be linking an entity to a player.

For example, let's create a dummy scoreboard for player/entity IDs.

    # In chat / load function
    scoreboard objectives add ID dummy

Now, if you are using command blocks, you can use this command to give each player a unique score in the `ID` scoreboard.

    # Command block
    execute as @a unless score @s ID = @s ID store result score @s ID run scoreboard players add #new ID 1

This command selects all players who do not have a score `ID`, then increases the score for [fake player](/wiki/questions/fakeplayer) `#new` in `ID` score by 1 and store this result to the player's `ID` score.

If you are using a datapack, then you can use the command above in the tick function, or create a simple advancement that will only run once for each player:

    # advancement example:first_join
    {
      "criteria": {
        "requirement": {
          "trigger": "minecraft:tick"
        }
      },
      "rewards": {
        "function": "example:set_id"
      }
    }
    
    # function example:set_id
    execute store result score @s ID run scoreboard players add #new ID 1

**And we're done, every player has a unique ID**. Now we can just copy the ID score to whatever entity we want to link up using `scoreboard players operation`, and use [this method](/wiki/questions/findsamescoreentity#method-2-store-the-score-in-a-fake-player-first) to find the entity with the same score (aka the linked entity).

***

> [!NOTE]
> The information below is outdated / inefficient. Use this only to better understand how the Scoreboard ID system works.

First we need to set up a dummy scoreboard objective

    scoreboard objectives add id dummy

Next, to make sure that every player gets a unique id, we need a system that assigns every player a unique score. This can be achieved by simply counting up with every subsequent player that needs a score. For this, we'll set a [fake player](/wiki/questions/fakeplayer) score of this objective to 1 to start with.

    scoreboard players set $total id 1

Next, to assign a unique id to every player that does not have an ID yet, it's best to use a function so that every new player get treated individually.

    # add 0 to all players' id score. This ensures that every player 
    # has a score (of 0) while not changing any existing scores
    scoreboard players add @a id 0

    # Execute as all players that have a score of 0 (aka no ID yet)
    execute as @a[scores={id=0}] run function namespace:init_id

`init_id.mcfunction`  

    # copy the score of the fake player to the player
    scoreboard players operation @s id = $total id

    # count up the score so the next player will get a different id
    scoreboard players add $total id 1

If for some reason you cannot use a function, use this code for your commandblocks instead. This means that it will take n ticks for n amount of players that need an id, while the other system is instant:

    scoreboard players add @a id 0

    # select one random unassigned player to assign the id to
    tag @r[scores={id=0}] add addId
    
    # apply id to this selected player, as above
    scoreboard players operation @a[tag=addId] id = $total id
    execute as @a[tag=addId] run scoreboard players add $total id 1

    # remove tag so we're ready for the next player
    tag @a remove addId

**And we're done, every player has a unique ID**. Now we can just copy the id score to whatever entity we want to link up using `scoreboard players operation`, and use [this method](/wiki/questions/findsamescoreentity) to find the entity with the same score (aka the linked entity).
