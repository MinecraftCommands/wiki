
# Add a delay to a command block

## Bedrock

In the Bedrock edition of the game, you can just give a commandblock a delay direcly through its interface, no need for any workarounds.

## Java 

Using repeaters with command blocks is considered bad practice, and is often impractical for long delays. Alternatives include area effect cloud timers, scoreboard timers and the `schedule` command (requires functions). They all have their respective use cases, but the most commonly used one is the scoreboard timer.

### Area Effect Clouds

Area effect clouds (AECs) have a `Duration` tag and an `Age` tag. Their `Age` will increase at a rate of 1 per tick. When its `Age` gets to its `Duration`, the AEC will disappear.

For example, the following creates an AEC that will disappear in 100 ticks (5 seconds), as their `Duration` defaults to 0:

    summon area_effect_cloud ~ ~ ~ {Age:-100}

We can set its `Particle` tag to `take` to prevent it from creating particles. We can also summon it with a scoreboard tag to select it with later:

    summon area_effect_cloud ~ ~ ~ {Age:-100,Particle:"take",Tags:["delay"]} 

The following 3 commands, in this order on a repeating-chain clock, will tag any AEC that's reached `Age:-1` with `delayActivating`, then make them activate (by turning on/off) an impulse command block at their position:

    tag @e[type=area_effect_cloud,tag=delay,nbt={Age:-1}] add delayActivating
    execute at @e[type=area_effect_cloud,tag=delayActivating] run data merge block ~ ~ ~ {auto:1b}
    execute at @e[type=area_effect_cloud,tag=delayActivating] run data merge block ~ ~ ~ {auto:0b}

That means that if you have those 3 commands repeating somewhere in your world, the command `/summon area_effect_cloud X Y Z {Age:-##,Particle:"take",Tags:[delay]}` can be used to activate the command block at (X, Y, Z) in ## ticks. An example, that'll activate the block at (10, 64, 30) in 70 ticks:

    summon area_effect_cloud 10 64 30 {Age:-70,Particle:"take",Tags:["delay"]}

### Scoreboard

For a scoreboard timer you can have a repeating commandblock somewhere that's counting up/down in a particular scoreboard objective and then use `execute if score` in the commandblock that should have the delay. You can either use individual player scores (recommended for player dependant events/delays) or "[fake player](/wiki/questions/fakeplayer)" scores (set "fake" values for player names, recommended for player independant delays).

    # the repeating commandblock
    scoreboard players add @a timer 1

    # the commandblock that should run for every player with a timer score of 100
    execute as @a[scores={timer=100}] run say this command has 5 seconds delay

    # alternatively for a fake player timer
    scoreboard players add $FakePlayer timer 1
    execute if score $FakePlayer timer matches 120 run say this command has 6 seconds delay


### Schedule

Using functions the schedule command can be used to make a function run in ## amount of ticks.

    schedule function <function: string>

This has several limitations:

1. Even when using `/execute as`, the scheduled function will always run as the Server.  
2. Scheduling the same function before it is successfully ran will by default overwrite the previous schedule: if you schedule a function to happen in 5 seconds, then schedule the same function again before the 5 seconds are up, the new schedule will be the one that happens. **Since 1.15 you can now add the `append` argument as the last argument in the command, which circumvents this problem**.  
3. It requires functions and thus datapacks to work.

See also: https://minecraft.wiki/Commands/schedule