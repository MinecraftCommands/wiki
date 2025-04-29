## Summon an entity at the position set in a score

| 📝 Note |
|---------|
|This article is about Java Edition, except for binary teleport, found at the end of the article, that will work in bedrock too|

| ⚠️ Important |
|--------------|
|If the coordinate is inside a block and below it there isn't a solid block, the entity will clip trought and fall, this is common if you are storing the scoreboard value by storing the `Pos` value, when the player is not in a full block (for example a slab)|

If you want to store the position of the player see this [transfer nbt to score](wiki/questions/nbttransfer)

Since version 1.20.2 you can summon the entity directly at the position of the score using a [macro](https://minecraft.wiki/w/Function_(Java_Edition)#Macros) in the datapack. If you are using an earlier version, or do not use a datapack, then you cannot summon the entity directly at the position of the score, instead you have to summon the entity and then teleport it to your desired position.

Below is an example of summoning a pig according to the set value in the scoreboard:

<details>
  <summary style="color: #e67e22; font-weight: bold;">See example</summary>

```mcfunction
# Setup
scoreboard objectives add pos dummy
scoreboard players set X pos 10
scoreboard players set Y pos 64
scoreboard players set Z pos 10
function example:summon/pig

# function example:summon/pig
execute store result storage example:macro pos.x int 1 run scoreboard players get X pos
execute store result storage example:macro pos.y int 1 run scoreboard players get Y pos
execute store result storage example:macro pos.z int 1 run scoreboard players get Z pos
function example:summon/pig_macro with storage example:macro pos

# function example:summon/pig_macro
$summon minecraft:pig $(x) $(y) $(z) {Tags:["custom_pig"]}
$particle minecraft:happy_villager $(x) $(y) $(z) 0.5 0.5 0.5 0 200
$tellraw @a "Pig summoning at $(x) $(y) $(z)"
```
</details>

**Note: You cannot use macro data in the same function in which you set data for the macro command. You should always run a separate function to execute the macro command.**

The macro allows you to insert any numeric or text data into any part of the command; however, before using these values in the command you need to set this data in storage, read the data from the entity / block, or you can manually set the values when running the function. Below is an example:

```mcfunction
# In chat
function example:macro_summon {id:"minecraft:pig",x:25.5d, y:65.5d, z:-15.5d}

# function example:macro_summon
$summon $(id) $(x) $(y) $(z)
```

## Teleport an entity to the position set in a score

Assuming your desired position is stored in the entity's posX, posY and posZ value, you can just run `execute store` to save the score into the Position like this:

```mcfunction
execute store result entity @s Pos[0] double 1 run scoreboard players get @s posX
execute store result entity @s Pos[1] double 1 run scoreboard players get @s posY
execute store result entity @s Pos[2] double 1 run scoreboard players get @s posZ
```

**Be aware that if the entity enters unloaded chunks at any point during this operation, it might not continue to work, especially if you're not using a function to do this**. So if your X moves the entity out of the loaded chunks, the Y and Z won't be applied anymore. If you're running the commands from above in a function you'll be fine, as we don't need to re-find the entity because we're using the `@s` selector.

But you can easily avoid unloading the entity if you do not change each axis separately, but first do it in storage and then copy the ready `Pos` tag from storage to the entity data:

```mcfunction
data merge storage example:data {Pos:[0d,0d,0d]}
execute store result storage example:data Pos[0] double 1 run scoreboard players get @s posX
execute store result storage example:data Pos[1] double 1 run scoreboard players get @s posY
execute store result storage example:data Pos[2] double 1 run scoreboard players get @s posZ
data modify entity @s Pos set from storage example:data Pos
```

## Teleport the player to the position set in a score

The problem with the player is that the player NBT data cannot be modified and thus we can't just set their position like we can with entities. There are four decent ways to go about this:

### 1: Macro function

Since version 1.20.2 you can also use the [macro](https://minecraft.wiki/w/Function_(Java_Edition)#Macros) to teleport to specified coordinates. Here is an example of running a macro function with data from `storage example:macro pos`:

```mcfunction
execute store result storage example:macro pos.x int 1 run scoreboard players get X pos
execute store result storage example:macro pos.y int 1 run scoreboard players get Y pos
execute store result storage example:macro pos.z int 1 run scoreboard players get Z pos
function example:tp/macro with storage example:macro pos

# function example:tp/macro
$tp @s $(x) $(y) $(z)
```

| 📝 Note |
|---------|
|A macro cannot be read from a List/Array, but only from an object. Therefore, if you received a list (like `Pos` tag) as a tag for teleportation, you must convert this into objects|

```mcfunction
# Example input pos as List
data merge storage example:data {Pos:[25d,100d,65d]}
function example:tp/convert

# function example:tp/convert
data modify storage example:macro pos.x set from storage example:data Pos[0]
data modify storage example:macro pos.y set from storage example:data Pos[1]
data modify storage example:macro pos.z set from storage example:data Pos[2]
function example:tp/macro with storage example:macro pos
```

### 2: End Gateways

You can use end gateways to teleport the players to an exact location, see [this for more info](https://minecraft.wiki/End_Gateway_(block)#Data_values).  

Basically you can set its block NBT Data to `ExactTeleport:1b,ExitPortal:{X:1,Y:2,X:3}` using `data merge block` or `execute store` and then teleport the player into said portal. Thanks to `execute store` you can set the Exit Portal NBT dynamically:

```mcfunction
# 1.13 - 1.20.4
setblock 0 5 0 minecraft:end_gateway{ExactTeleport:1b,ExitPortal:{X:0,Y:0,Z:0},Age:-9223372036854775808L}
execute store result block 0 5 0 ExitPortal.X int 1 run scoreboard players get @s MapX
execute store result block 0 5 0 ExitPortal.Y int 1 run scoreboard players get @s MapY
execute store result block 0 5 0 ExitPortal.Z int 1 run scoreboard players get @s MapZ
tp @s 0 5 0
```

Make sure the position you're using for the endgateway is in the loaded chunks. _If the `Age` tag is not set to a negative number, then End Gateways sometimes shoot out beacon beams, especially when they are newly created, so you might want to set the portal once and not every time you need it._

Since version 1.20.5, the `ExitPortal` Compound tag has been replaced by the [Int Array](https://minecraft.wiki/w/NBT_format#Data_types) `exit_portal` tag (not a List).

```mcfunction
# 1.20.5+
setblock 0 5 0 minecraft:end_gateway{ExactTeleport:1b,exit_portal:[I;0,0,0],Age:-9223372036854775808L}
execute store result block 0 5 0 exit_portal[0] int 1 run scoreboard players get @s MapX
execute store result block 0 5 0 exit_portal[1] int 1 run scoreboard players get @s MapY
execute store result block 0 5 0 exit_portal[2] int 1 run scoreboard players get @s MapZ
tp @s 0 5 0
```

### 3: Use an entity

Summon an entity, use the above method to teleport the entity to the position and then teleport the player to the entity. Here it is also important that the entity is `@s` from the moment you move it, in case it goes into unloaded chunks.  
See [u/SanianCreations](https://www.reddit.com/u/SanianCreations) post about this [here](https://www.reddit.com/r/MinecraftCommands/comments/fd1lds/new_method_to_tp_to_scoreboard_values).

### 4: Binary Teleportation

| 📝 Note |
|---------|
|This method works in bedrock too|

You basically copy their score to some temporary score so you don't lose it when you modify it, and then you go through the different powers of 2 (hence the name), check if their score is above that and then teleport them relatively that far.

<details>
  <summary style="color: #e67e22; font-weight: bold;">See example</summary>

```mcfunction
execute as @a unless score @s xTmp matches 0 if score @s xTmp = @s xTmp run teleport @s 0 0 0

execute as @a[scores={xTmp=128..}] at @s run tp @s ~128 ~ ~
scoreboard players remove @a[scores={xTmp=128..}] xTmp 128

execute as @a[scores={xTmp=64..}] at @s run tp @s ~64 ~ ~
scoreboard players remove @a[scores={xTmp=64..}] xTmp 64

execute as @a[scores={xTmp=32..}] at @s run tp @s ~32 ~ ~
scoreboard players remove @a[scores={xTmp=32..}] xTmp 32

....
all the way down to 1, repeat for all 3 coordinates
```
</details>

This does use a lot of commands but using a function it's fairly easy to do and you can go as high as you want. Always start with the highest power of 2.
