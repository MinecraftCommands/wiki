## Summon an entity at the position set in a score

You cannot summon the entity directly at the position of the score, instead you have to summon the entity and then teleport it to your desired position.

## Teleport an entity to the position set in a score

Assuming your desired position is stored in the entity's posX, posY and posZ value, you can just run `execute store` to save the score into the Position like this:

    execute store result entity @s Pos[0] double 1 run scoreboard players get @s posX
    execute store result entity @s Pos[1] double 1 run scoreboard players get @s posY
    execute store result entity @s Pos[2] double 1 run scoreboard players get @s posZ

**Be aware that if the entity enters unloaded chunks at any point during this operation, it might not continue to work, especially if you're not using a function to do this**. So if your X moves the entity out of the loaded chunks, the Y and Z won't be applied anymore. If you're running the commands from above in a function you'll be fine, as we don't need to re-find the entity because we're using the `@s` selector.

## Teleport the player to the position set in a score

The problem with the player is that the player NBT data cannot be modified and thus we can't just set their position like we can with entities. There are three decent ways to go about this:

### 1: Binary Teleportation

you basically copy their score to some temporary score so you don't loose it when you modify it, and then you go through the different powers of 2 (hence the name), check if their score is above that and then teleport them relatively that far.

    teleport @p 0 0 0
    
    execute at @p if entity @p[scores={xTmp=128..}] tp @p ~128 ~ ~
    scoreboard players remove @p[scores={xTmp=128..}] xTmp 128
    
    execute at @p if entity @p[scores={xTmp=64..}] tp @p ~64 ~ ~
    scoreboard players remove @p[scores={xTmp=64..}] xTmp 64
    
    execute at @p if entity @p[scores={xTmp=32..}] tp @p ~32 ~ ~
    scoreboard players remove @p[scores={xTmp=32..}] xTmp 32
    
    ....
    all the way down to 1, repeat for all 3 coordinates

this does use a lot of commands but using a function it's fairly easy to do and you can go as high as you want. Always start with the highest power of 2.

### 2: End Gateways

You can use end gateways to teleport the players to an exact location, see [this for more info.](https://minecraft.wiki/End_Gateway_(block\)#Data_values).  

Basically you can set its block NBT Data to `ExactTeleport:1b,ExitPortal:{X:1,Y:2,X:3}` using `data merge block` or `execute store` and then teleport the player into said portal. Thanks to `execute store` you can set the Exit Portal NBT dynamically:

    setblock 0 5 0 minecraft:end_gateway{ExactTeleport:1b,ExitPortal:{X:0,Y:0,Z:0}}
    execute store result block 0 5 0 ExitPortal.X int 1 run scoreboard players get @s MapX
    execute store result block 0 5 0 ExitPortal.Y int 1 run scoreboard players get @s MapY
    execute store result block 0 5 0 ExitPortal.Z int 1 run scoreboard players get @s MapZ
    tp @s 0 5 0

Make sure the position you're using for the endgateway is in the loaded chunks. _End Gateways sometimes shoot out beacon beams, especially when they are newly created, so you migh want to add set the portal once and not every time you need it._

### 3: Use an entity

Summon an entity, use the above method to teleport the entity to the position and then teleport the player to the entity. Here it is also important that the entity is @s from the moment you move it, in case it goes into unloaded chunks.  
See /u/SanianCreations' post about this [here](https://www.reddit.com/r/MinecraftCommands/comments/fd1lds/new_method_to_tp_to_scoreboard_values).