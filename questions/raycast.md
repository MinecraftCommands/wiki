# Raycast

> [!NOTE]
> If you're trying to check whether an entity / player is looking at a specific entity / position, you might be able to use a more streamlined method [described here](/wiki/questions/lookat). 

## What is a raycast?

Raycasting (or more accurately in this context: ray marching) in minecraft commands describes a process of moving forward in small increments (basically "drawing" a line) until a certain condition is met. It has many possible usecases, most often it is used to check what a player/entity is looking at by following the looking direction.

A raycast is possible due to the introduction of [local coordinates](https://minecraft.wiki/wiki/Coordinates#Local_coordinates) (`^ ^ ^`) which allows a commands origin to be moved relative to the entities local coordinates instead of absolute or world based relative coordinates.

In all of the below examples we'll use a step-size of 0.1 blocks at a time. This is a reasonable compromise between error margin and performance hit, but depending on the usecase a smaller or larger step size is absolutely thinkable.

Due to the way these raycasts tend to be implemented, they move forward until they hit the desired target (e.g. a block that is not air, but you may also want to not detect other non-solid blocks as water). However, this way it is possible that a ray will never find its terminating condition, so a second ending condition is added: The maximum step amount. This generally consists of a scoreboard value that is increased/decreased with every step, counting the number of steps thus stopping the ray after a certain distance.

Please remember that a raycast _can_ (depending on the exact implementation) be a somewhat performance intensive procedure, so you need to try to find a balance between accuracy, distance and frequency of your raycast to avoid lag.

-----

## Java

You can generate a raycasting datapack using this generator: https://sourceblock.net/beta/en-US/tools/data-packs/raycasting-generator

### Without an entity

This is the preferred method of raycasting, as this entityless approach causes less strain on the server and you don't need to clean up the used entity afterwards. Of course you can still summon an entity to mark the position you've found for later useage, but it is often encouraged to do it all in a single tick.

This method only works in a single tick, so if you need the raycast to be over a period of time instead of instantaneous, use the entity based method instead. It also requires functions to work, so if you're unable to use a datapack, you also need to use the other method.

First, the command that will initiate a raycast. We're anchoring the execution point at the entities eyes (moving up from their feet), changing the execution position to the eye level and then, due to a quirk in minecrafts keeping of context, move the anchor back to the feet to avoid the game reapplying the hight of the eyes to every step.

    execute as <shooter> at @s anchored eyes positioned ^ ^ ^ anchored feet run function namespace:start_ray

Next up, the ray setup. We need to set a maxium amount of steps that we can go before we abort our search. This means we can just set a score to the maximum amount of steps. Here a `dummy` scoreboard named `ray_steps` is used. It's set to 50 steps, so a maximum distance of 5 blocks considering our step distance is 0.1 blocks per step (which is also the reach distance of a player). We also want to store whether our ray successfully hit something, so we know whether to continue or to stop. In this case another `dummy` objective is used, called `ray_success`. Instead of two scoreboards you can also use a single scoreboard and use [fake players](/wiki/questions/fakeplayer) to store the scores.

`namespace:start_ray`

    scoreboard players set @s ray_steps 50
    scoreboard players set @s ray_success 0
    function namespace:ray

Next, the actual ray is being cast. For that, we first check whether our stopping condition has been achieved and run a success function if we did. Next we count up the steps we can still take by 1. And lastly we run the function again, moved forward by our step size, if we neither hit the stopping condition nor the maximum step size. In this example the stopping condition is hitting a block that is not air.

`namespace:ray`

    execute unless block ~ ~ ~ minecraft:air run function namespace:hit_block
    scoreboard players remove @s ray_steps 1
    execute if score @s ray_steps matches 1.. if score @s ray_success matches 0 positioned ^ ^ ^0.1 run function namespace:ray

Lastly we can use the success function to run whatever we intend to do at the found place. In this case we'll just set a stone block. _Make sure to set the `ray_success` score to 1 at some point in the function though!_

`namespace:hit_block`

    scoreboard players set @s ray_success 1
    setblock ~ ~ ~ stone

And that's the basic skeleton of a raycast, which can now be extended to your hearts content.

### With an entity

Please see the [without an entity](#wiki_without_an_entity) section above for a step by step explanation, as in this section only the differences will be highlighted.

The main difference being that we are now moving an entity instead of just the execution context, so we also need to create said entity.

`namespace:start_ray`

    # summon entity to use as a marker
    summon area_effect_cloud ~ ~ ~ {Tags:["ray_marker"]}
    # make sure the marker entity is rotated in the same way as the executing entity
    tp @e[tag=ray_marker] ~ ~ ~ ~ ~
    
    scoreboard players set @e[tag=ray_marker] ray_steps 50
    scoreboard players set @e[tag=ray_marker] ray_success 0
    execute as @e[tag=ray_marker] at @s run function namespace:ray

We are now executing the ray function as the marker entity instead of the executing entity. So, to move the execution position, we need to teleport the entity and make sure we re-align the function position to the entity at every step, instead of just repositioning the execution context.

`namespace:ray` 

    execute unless block ~ ~ ~ minecraft:air run function namespace:hit_block
    scoreboard players remove @s ray_steps 1
    tp @s ^ ^ ^0.1
    execute if score @s ray_steps matches 1.. if score @s ray_success matches 0 at @s run function namespace:ray

Make sure you kill the entity once it's done it's job, for example by putting the kill command into the success function or at the end of the `start_ray` function.

If you're doing this **without a datapack**, just using commands, then you just need to make sure all the commands that are in the `ray` function are executed as and at the ray_marker entity, that the commands in the `start_ray` function are executed as the ray shooter entity and that you're only affecting the one ray_marker that was summoned on that entity.

-----

## Bedrock

### Without an entity

Thanks to the new execute syntax introduced in 1.19.50, you can use the same system as described in Java - without an entity.

### With an entity

#### Do it all in one tick with commands (using XP orbs)

A technique first brought up by [/u/VentedMCBE](https://www.reddit.com/u/VentedMCBE) allows us to do a full raycast using only commandblocks and doing it in a single tick. It is, as most advanced minecraft techniques, somewhat hacky, but it works well.

As mentioned, this works with just commandblocks, so for the following commands you can either put them into a repeating-chain of commandblocks or in a function.

    # ignore non-specifically summoned xp orbs
    tag @e[type=xp_orb] add ignore
    # Summon ray marker per player
    execute as @a at @s run summon xp_orb
    # Rotate and relocate the ray marker
    execute as @e[type=xp_orb] at @s at @p rotated as @p anchored eyes run tp @s ~~~ ~ ~
    # Move the ray forwards
    execute as @e[c=2] as @e[c=2] as @e[c=2] as @e[c=2] as @e[c=2] as @e[c=2] as @e[c=2] as @e[c=2] as @e[c=2] as @e[type=xp_orb,tag=!ignore] at @s run tp @s ^^^0.1 true
    # Run a command or display ray end point
    execute at @e[type=xp_orb,tag=!ignore] run particle minecraft:basic_crit_particle ~~~
    # Kill ray orbs
    kill @e[type=xp_orb,tag=!ignore]

The 4th command is where the magic happens: Due to us splitting the execution path into 2, every time `as @e[c=2]` is called, we're essentially doubling the executions over and over. So because we're calling it 9 times, the rest of the command will be run 2^9 = 512 times. You can adjust this accordingly to your needs. In this case we're moving the ray forwards by 0.1 blocks (thus to a maximum of 51.2 blocks), only if we can move into the block there (as denoted by the `true` at the end of the tp command). Thus this command would cause us to find the next solid block the player is looking at (in a 51 block radius). 

So, for different applications you'd modify the 4th command to find different things. For example, to stop moving if there is a creeper (feet) close by, going through blocks, the command could look like this:

    execute as @e[c=2] ... as @e[c=2] as @e[type=xp_orb,tag=!ignore] at @s positioned ^^^0.1 unless entity @e[type=creeper,r=1] at @s run tp @s ^^^0.1

#### Using functions

This is the same method as described in the "[with an entity](#wiki_with_an_entity)" section above, just using bedrock syntax. Please see above for an explanation on what is going on.

> **Please note that there seems to be a limitation in bedrock at the time of writing (1.18) that limits the entities up/down movement to 45° angles or just straight up 0°!** Thus a different method might need to be found, possibly using custom projectiles.

It is _highly_ recommended to use a custom entity for the raycast, as you not only want it to be invisible, without any AI but also unaffected by gravity.

Command to start the raycast (the execution point is moved up manually towards player standing eye level. This means that if the player is not standing up normally (e.g. crouching or swimming) this will produce unexpected results):

    execute <shooter> ~ ~1.62 ~ function namespace:start_ray

`namespace:start_ray`

    # summon entity to use as a marker
    summon custom:entity ray_marker ~ ~ ~
    # make sure the marker entity is rotated in the same way as the executing entity
    tp @e[name=ray_marker] ^ ^ ^-0.1 facing @s
    
    scoreboard players set @e[name=ray_marker] ray_steps 50
    scoreboard players set @e[name=ray_marker] ray_success 0
    execute @e[name=ray_marker] ~ ~ ~ function namespace:ray

checking for all but one block needs a few more steps in bedrock, so in this example we're instead checking for the block being grass.

`namespace:ray` 

    execute @s ~ ~ ~ detect ~ ~ ~ grass function namespace:hit_block
    scoreboard players remove @s ray_steps 1
    tp @s ^ ^ ^0.1
    execute @s[scores={ray_steps=1..,ray_success=0}] ~ ~ ~ function namespace:ray
