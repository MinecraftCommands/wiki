
# Add a delay to a command block

## Bedrock

In the Bedrock edition of the game, you can just give a commandblock a delay direcly through its interface, no need for any workarounds, unless you want to make a timer for every player, see the method using a [scoreboard](wiki/questions/blockdelay.md#scoreboard)

## Java 

Using repeaters with command blocks (or a hopper clock) is considered bad practice, it causes more lag, and is often impractical for long delays. Alternatives include area effect cloud timers, scoreboard timers and the `schedule` command (requires functions). They all have their respective use cases, but the most commonly used one is the scoreboard timer.

### Area Effect Clouds

Area effect clouds (AECs) have a `Duration` tag and an `Age` tag. Their `Age` will increase at a rate of 1 per tick. When its `Age` gets to its `Duration`, the AEC will disappear.

For example, the following creates an AEC that will disappear in 100 ticks (5 seconds), as their `Age` defaults to 0:

    summon area_effect_cloud ~ ~ ~ {Duration:100}

> [!NOTE]
> To see area_effect_cloud press `F3 + B` to show hitboxes.

You can set the `Duration` tag as a positive value or the `Age` tag as a negative value, but then you need to set the `Particle` tag to `"block air"` (1.20.4 and below) or `{type:"block",block_state:"minecraft:air"}` (1.20.5 and above) to prevent it from creating particles.

We can also summon it with a tag to select it with later:

    # 1.20.4 and below
    summon area_effect_cloud ~ ~ ~ {Tags:["delay"],Age:-100,Particle:"block air"} 

    # 1.20.5 and above
    summon area_effect_cloud ~ ~ ~ {Tags:["delay"],Age:-100,Particle:{type:"block",block_state:"minecraft:air"}} 

The following 3 commands, in this order on a repeating-chain clock, will tag any AEC that's reached `Age:-1` with `delayActivating`, then make them activate (by turning on/off) an impulse command block at their position:

    tag @e[type=area_effect_cloud,tag=delay,nbt={Age:-1}] add delayActivating
    execute at @e[type=area_effect_cloud,tag=delayActivating] run data merge block ~ ~ ~ {auto:true}
    execute at @e[type=area_effect_cloud,tag=delayActivating] run data merge block ~ ~ ~ {auto:false}

That means that if you have those 3 commands repeating somewhere in your world, the command `/summon area_effect_cloud X Y Z {Tags:[delay],Age:-##,Particle:{type:"block",block_state:"minecraft:air"}}` can be used to activate the command block at (X, Y, Z) in ## ticks. An example, that'll activate the block at (10, 64, 30) in 70 ticks:

    summon area_effect_cloud 10 64 30 {Tags:["delay"],Age:-70,Particle:{type:"block",block_state:"minecraft:air"}}

However, if you want to use spawn_egg to simply create an AEC's for delay, then you need to add Radius and WaitTime:

    # 1.13 - 1.20.4
    give @s bat_spawn_egg{EntityTag:{id:"minecraft:area_effect_cloud",Tags:["delay"],Duration:100,Radius:0f,WaitTime:0}}
    
    # 1.20.5+
    give @s bat_spawn_egg[entity_data={id:"minecraft:area_effect_cloud",Tags:["delay"],Duration:100,Radius:0f,WaitTime:0}]

> [!NOTE]
> You can use any `spawn_egg`, but not just `bat_spawn_egg`

    # Command block / tick function
    execute at @e[type=area_effect_cloud,tag=delay,nbt={Age:99}] run summon zombie

This is a simple way to execute any command at a specified position once with a specified delay.

### Marker

If you need to execute a command not only once, but every 5 seconds, for example, at specific location, then you can use the [marker entity](https://minecraft.wiki/w/Marker) (1.17+) for this. If you are on an earlier version use an [`area_effect_cloud`](https://minecraft.wiki/w/Lingering_Potion#ID_3) that will not despawn or an invisible [`armor_stand`](https://minecraft.wiki/w/Armor_Stand).

    # Summon
    summon marker ~ ~ ~ {Tags:["delay"]}
    
    # Spawn egg
    ## 1.17 - 1.20.4
    give @s bat_spawn_egg{EntityTag:{id:"minecraft:marker",Tags:["delay"]}}
    
    ## 1.20.5+
    give @s bat_spawn_egg[entity_data={id:"minecraft:marker",Tags:["delay"]}]

In addition to the marker, you need to use a scoreboard timer, which each tick will add 1 to the score marker.

    # Command blocks / tick function
    scoreboard players add @e[type=marker,tag=delay] timer 1
    execute as @e[type=marker,tag=delay,scores={delay=100..}] at @s store success score @s delay run summon zombie

### Scoreboard

For a scoreboard timer you can have a repeating commandblock somewhere that's counting up/down in a particular scoreboard objective and then use `execute if score` in the commandblock that should have the delay. You can either use individual player scores (recommended for player dependant events/delays) or "[fake player](/wiki/questions/fakeplayer)" scores (set "fake" values for player names, recommended for player independant delays).

    # Setup
    scoreboard objectives add timer dummy
    
    # The repeating command_block
    scoreboard players add @a timer 1

    # The command_block that should run for every player with a timer score of 100
    execute as @a[scores={timer=100}] run say This command has 5 seconds delay.
    scoreboard players reset @a[scores={timer=100..}] timer

    # Alternatively for a fake player timer
    scoreboard players add $FakePlayer timer 1
    execute if score $FakePlayer timer matches 120 run say This command has 6 seconds delay.
    execute if score $FakePlayer timer matches 120.. run scoreboard players reset $FakePlayer timer

Or, if you do not create additional conditions, you can immediately reset the score in one command using `store success score` (only java edition):

    # Command blocks
    execute as @a[scores={timer=101..}] store success score @s timer run say This command has 5 seconds delay.
    execute if score $FakePlayer timer matches 121.. store success score $FakePlayer timer run say This command has 6 seconds delay.

This command will not only execute the command, it also works as a timer reset command, however this implementation may not have the exact delay in some cases where your command is executed in some conditions and not in another.

You can also make the delay more dynamic by setting a score of a fake player and comparing it to the players score:

    # Set delay
    scoreboard players set #delay timer 300
    
    # Command block / tick function
    execute as @a if score @s timer >= #delay timer store success score @s timer run say Custom delay command.

### Schedule

Using functions the [schedule command](https://minecraft.wiki/Commands/schedule) can be used to make a function run in ## amount of ticks.

    schedule function <function> <time> [append|replace]

So you can create a simple way to run your commands not every tick, but, for example, once a second:

    # function example:load
    function example:loops/10s
    
    # function example:loops/10s
    schedule function example:loops/10s 10s
    say This will run every 10 second.

> [!NOTE]
> Do not run the schedule function in a tick function, without any conditions. This will overwrite the schedule every tick and the schedule function will never run.

This has several limitations:

1. Even when using `/execute as`, the scheduled function will always run as the Server, but not as the selected entity. See [command context](wiki/questions/commandcontext)
2. Scheduling the same function before it is successfully ran will by default overwrite the previous schedule: if you schedule a function to happen in 5 seconds, then schedule the same function again before the 5 seconds are up, the new schedule will be the one that happens. **Since 1.15 you can now add the `append` argument as the last argument in the command, which circumvents this problem**.  
3. It requires functions and thus datapacks to work.
4. It will be executed at position Y = -64 under the world spawn.
5. It will stop the scheduled function when (re)loading the world

```
# In chat
execute as @a run schedule function example:some_function 5s

# function example:some_function
say This is a message from the server.
```

If you want to schedule a function to execute as an entity, here is a method that allows you to do that with different schedules for different entities:

Read the current gametime and store it in the score of the selected entity and add your delay to this score. Then run the schedule function and count the gametime again and find the entities with the same score value.

    # Run shedule function (as entity)
    execute store result score @s timer run time query gametime
    scoreboard players add @s timer 150
    schedule function example:delay_function 150t append
    
    # function example:delay_function
    execute store result score #this timer run time query gametime
    execute as @e if score @s timer = #this timer run say Example Command.

> [!NOTE]
> If you frequently run the schedule function to delay, then use `append` mode to run the schedule function so that each run does not overwrite the previous one.
