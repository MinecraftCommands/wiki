# Shoot a Projectile/Entity in the direction a player is facing

*Java only. This doesn't work in bedrock and there are currently no known workarounds.*

***Advanced**. It is highly recommended that you use functions for this and already know how `execute at` and `execute as` works!*

Shooting a projectile/entity in the direction a player is facing is not trivial, and in the older versions of the game, the process was to basically hardcode a bunch of possible motions for various facing values of the player. That technique however is completely obsolete as of 1.13.  

*This article has been updated to use a `marker` entity. If you're using an older version that doesn't have the marker entity yet, swap it out for an `area_effect_cloud`.*

*Related: [How to detect item click](/wiki/questions/itemclick)*

## Using teleports to essentially do vector calculations

### Explanation

Sounds complicated, but is actually pretty easy. The end goal is to find the correct Motion values to apply to the entity for it to fly into the correct direction. Motion is a three dimensional vector, where each axis corresponds to one of the ingame directions: `Motion:[xMotion,yMotion,zMotion]`. Each value is limited to [-10..10] and will be clamped to these values if you go above or below this limit. A value of 1 means a motion of 1 block per tick in that axis positive direction. Motion will also automatically be changed by the games physics engine, so we only need to set the initial motion and the rest will happen by itself.

To find the correct Motion to apply, we need to find the difference between the position the entity starts in and the position the entity should be in the next tick after that (because that's how far it's supposed to move in this one tick). Luckily,  we can get both of these positions from the local coordinates (`^ ^ ^`) of the shooting player. For the following examples we'll assume the entity should be shot out with a speed of 1 block per tick in the direction the player is facing (`^ ^ ^1`), but this can be altered to your liking (but remember the limit of 10 in the motion tag!).  

So, we can get the players location using its `Pos` NBT values and we can summon an entity `^ ^ ^1` blocks in front of the player, using its `Pos` attribute to get the second position and thus we know starting position P and target position T. From here vector calculations tell us that to get the movement vector V between those two  points, all we need to do is T - P = V for each value individually (T.x - P.x = V.x etc.). Then we have V and can store it into the Motion values of our entity.

-----

## Method 1: Let the game do the math for us by using the zero point (simplified)

Or rather, make use of the fact that we can use the worlds zero position and pretend the player is there. And because X - 0 = X, we don't need to do the math ourselves but can directly store the result into the entity. Here is how it works (running `as` and `at` the player):

    # summon temporary entity "in front of the player", if the player was standing at 0 0 0
    execute positioned 0.0 0 0.0 run summon marker ^ ^ ^1 {Tags:["direction"]}

    # summon the projectile entity (e.g. sheep, but can also be an arrow/snowball/etc. 
    # When using a projectile, you might want to summon it in front of the player so it doesn't hit the player themselves)
    # you might want to summon it at the players eyes as well using anchored eyes
    summon sheep ~ ~ ~ {Tags:["projectile"]}

    # copy the markers position tag to the sheeps motion tag
    data modify entity @e[type=sheep,tag=projectile,limit=1] Motion set from entity @e[type=marker,tag=direction,limit=1] Pos

    # clean up
    tag @e[tag=projectile] remove projectile
    kill @e[tag=direction]

**This method requires the worlds `0 0 0` point to be loaded to work. If that is not the case or cannot be guaranteed, use the following modification instead (still `as` and `at` the player).** It ensures that the marker entity doesn't get lost along the way by being teleported to unloaded chunks. Luckily the `@s` selector is able to still select the entity even if it was unloaded, which means we can do it as follows.

    # summon the temporary entity at the players position
    summon marker ~ ~ ~ {Tags:["direction"]}
    
    # execute the below function as the marker entity, so it doesn't get lost from being unloaded
    # also run positioned at the world zero point
    execute as @e[tag=direction,limit=1] positioned 0.0 0.0 0.0 run function namespace:get_motion
    
    # summon the projectile entity. Again, it might make sense to summon the projectile at the players eyes
    # and in front of them, so we'll do that in this example
    execute anchored eyes run summon sheep ^ ^ ^1 {Tags:["projectile"]}
    
    # store the previously stored Motion into the projectile entity
    data modify entity @e[tag=projectile,limit=1] Motion set from storage namespace:storage Motion
    
    # clean up the tag
    tag @e[tag=projectile] remove projectile

`get_motion.mcfunction`

    # this function is executed as the marker entity, positioned at 0 0 0 and still rotated as the player
    # (as that wasn't changed with the function call)
    
    # teleport the entity forward by 1 block (based on the player rotation and the position 0 0 0).
    tp @s ^ ^ ^1
    
    # store the current position in the worlds NBT storage so we don't loose it
    data modify storage namespace:storage Motion set from entity @s Pos
    
    # we don't need this entity anymore
    kill @s

Since version 1.19.4 you can use `summon` inside the `/execute` command, which allows you to simplify shootfacing and combine the command of calling the direction of an entity and modifying the `Motion` tag of the projectile.

Now, instead of a marker that must be manually killed, you can use area_effect_cloud, which will disappear on its own on the next tick. Also, force loaded coordinates 0 0 0 are no longer required to work, since we are creating and using direction entities within one command, but it is still strongly recommended to load chunks [-1, -1] to [0, 0], since area_effect_cloud will not be deleted in unloaded chunks automatically.

    # Summon the projectile entity
    summon sheep ~ ~ ~ {Tags:["projectile"]}
    
    # Use player rotation to create an area_effect_cloud of about 0 0 and immediately copy the position of this entity into the projectile motion tag.
    execute rotated as <player> positioned 0.0 0.0 0.0 positioned ^ ^ ^1 summon area_effect_cloud run data modify entity @e[tag=projectile,limit=1] Motion set from entity @s Pos
    
    # Remove projectile tag
    tag @e[tag=projectile] remove projectile

`^ ^ ^1` is used as the throw strength.

You can go ahead and do this using almost just one command block. You can perform all settings using `store success entity`. For example, for a fireball, you can immediately set ExplosionPower and power[1] so that the projectile does not fly in a straight line, but in an arc:

    # Command blocks
    tag @e[type=fireball,tag=!exist] add exist
    execute as <player> at @s anchored eyes positioned ^ ^ ^3 summon fireball store success entity @s ExplosionPower byte 4 store success entity @s power[1] double -0.08 store success score @s fix positioned 0.0 0.0 0.0 positioned ^ ^ ^1 summon area_effect_cloud run data modify entity @e[type=fireball,tag=!exist,limit=1] Motion set from entity @s Pos

However, you may notice that this projectile lags in flight. Read about how to fix this in the [[Visual bug fix]](/questions/shootfacing.md#visual-bug-fix) section.

-----

## Method 2: Storing all values (for easier manipulation)

*Please note that you probably shouldn't be using this method if method 1 covers your usecase.*

This example will assume you have a dummy scoreboard objective set up that is named `pos` and that you're running these commands `as` and `at` the player (in a function). Whenever you see `#<something>` this is a [fake player](/wiki/questions/fakeplayer) that we use for convenient value storage. Because scoreboards can only hold integer values (whole numbers), we scale up the values by 1000 when storing them and down by 0.001 when putting them back into the Motion tag so we don't lose too much accuracy. We need to use scoreboards however, because we need to do some subtractions which is only conveniently possible with scoreboard operations. (This means that the system will stop working once you go above a position of 2'000'000 on either axis, if that is a concern, change it to 100/0.01, that way you can reach 2/3 to the worldborder without issues.)

    # summon the temporary entity
    summon marker ^ ^ ^1 {Tags:["direction"]}
    
    # get the coordinates of the player and the entity
    execute store result score #playerX pos run data get entity @s Pos[0] 1000
    execute store result score #playerY pos run data get entity @s Pos[1] 1000
    execute store result score #playerZ pos run data get entity @s Pos[2] 1000
    execute store result score #targetX pos as @e[type=marker,tag=direction,limit=1] run data get entity @s Pos[0] 1000
    execute store result score #targetY pos as @e[type=marker,tag=direction,limit=1] run data get entity @s Pos[1] 1000
    execute store result score #targetZ pos as @e[type=marker,tag=direction,limit=1] run data get entity @s Pos[2] 1000
    
    # do the math
    scoreboard players operation #targetX pos -= #playerX pos
    scoreboard players operation #targetY pos -= #playerY pos
    scoreboard players operation #targetZ pos -= #playerZ pos

    # summon the projectile entity
    summon sheep ~ ~ ~ {Tags:["projectile"]}

    # apply motion to projectile
    execute store result entity @e[type=sheep,tag=projectile,limit=1] Motion[0] double 0.001 run scoreboard players get #targetX pos
    execute store result entity @e[type=sheep,tag=projectile,limit=1] Motion[1] double 0.001 run scoreboard players get #targetY pos
    execute store result entity @e[type=sheep,tag=projectile,limit=1] Motion[2] double 0.001 run scoreboard players get #targetZ pos

    # clean up, ready for the next player
    tag @e[tag=projectile] remove projectile
    kill @e[tag=direction]

Instead of using 2 entities, you can also only use the projectile entity. Make sure if you teleport it twice (once to the "target" position and then to the "starting" position) that you apply the motion **after** the teleportation, because teleportation resets y motion.

Also, using this Method instead of Method 1, you can more easily modify the individual values (say you want X to be halved, y to be doubled and z should always get 1 added, for some reason, then you can easily do that). Apply your modifications after the `#do the math` part.

----

## Visual bug fix

However, you may notice that some entities that you want to use may visually lag when summoned and fall in front of the player. This is due to the fact that the movement of this entity is not updated on the client, but this can be easily fixed by updating the NBT data **on the next tick** in any way. The simplest and “safest” thing is to update the `Air` store tag with some value.

    execute as @e[tag=projectile] store result entity @s Air short 1 run time query gametime

**It is important to update the data on the next tick, but not on the current one!**

If you are using command blocks, you can achieve this by running this command **before** summon and changing the `Motion` tag. Here is a complete example for command blocks:

    # In chat
    scoreboard objectives add click used:carrot_on_a_stick
    scoreboard objectives add fix dummy
    forceload add -1 -1 0 0
    
    # Command blocks
    execute as @e[tag=projectile,scores={fix=1}] store result entity @s Air short 1 run time query gametime
    tag @e[tag=projectile] remove projectile
    execute as @a[scores={click=1..}] at @s anchored eyes run summon snowball ^ ^ ^ {Tags:["projectile"]}
    execute rotated as @a[scores={click=1..}] positioned 0.0 0.0 0.0 positioned ^ ^ ^1 summon minecraft:area_effect_cloud store success score @s fix run data modify entity @e[tag=projectile,limit=1] Motion set from entity @s Pos
    scoreboard players reset @a click

When using a datapack, you can use the shedule function to do this fix.

    # function example:tick
    execute as @a[scores={click=1..}] at @s run function example:click
    
    # function example:click
    scoreboard players reset @s click
    execute anchored eyes positioned ^ ^ ^ summon snowball run function example:shootfacing
    
    # function example:shootfacing
    tag @s add this
    execute positioned 0.0 0.0 0.0 positioned ^ ^ ^1 summon minecraft:area_effect_cloud run data modify entity @e[tag=this,limit=1] Motion set from entity @s Pos
    tag @s remove this
    tag @s add fix
    schedule function example:fix 2t replace
    
    # function example:fix
    execute unless entity @s as @e[tag=fix] run function example:fix
    execute store result entity @s Air short 1 run time query gametime
    tag @s remove fix

You may notice that in the case of the datapack, the schedule is runs with a delay of 2 ticks, but not 1 tick. This is because inside a tick schedule, the tick function runs before the schedule function runs, and therefore a 1 tick delay when running from a tick function will run the function on the same tick, but not on the next one.
