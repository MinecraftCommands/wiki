# Optimising

*Relevant information on Functions are generally at the end of their paragraph.*

## Turn off commands when they don't need to be running

For example, if you have a function or chain of command blocks that control an arena, you probably only need those running when someone is actually in the arena.

Consider a "branching" structure of checks. You could start with just `/execute if entity @a` (Or `/testfor @a` in pre-1.13). If and only if that succeeds you'd run checks to do with players, such as whether a player is holding *any* special item. If a player is holding any special item, run checks to see what kind of special item it is. Only then off of these last checks would you run the commands that make that item work.

**Don't** achieve this by repeatedly using `/execute` as a condition (e.g: `/execute as @a[tag=holdItem] ...`) while still having all the blocks/commands running, as this will still need to repeatedly evaluate the selector for each command. Instead, you want the commands to not be running at all if the previous check failed.

One method to implement is with [repeating conditional blocks](http://i.imgur.com/fhyzMXI.png), which are simple to set up and will not cause any additional chunk updates, but may cause problems with physical space restraints and the layout of your command blocks. They will only run their own chain when the block in the chain behind them has succeeded.

You can alternatively use the following commands to activate an impulse command block at X Y Z, then reset it so that it can be activated again the next tick:

    # pre-1.13 syntax
    blockdata X Y Z {auto:1b}
    blockdata X Y Z {auto:0b}
    # 1.13+ syntax
    data merge block X Y Z {auto:1b}
    data merge block X Y Z {auto:0b}

These commands should be conditional blocks on the chain that performs the condition test. Your setup might look [something like this](http://i.imgur.com/vyhnmTH.png).

**Functions** make this even easier and more efficient. You can run a different function only if a condition (for example, if a specified selector exist) succeeds/fails, for example:

    # pre-1.13 syntax
    function code:arena_events if @a[tag=in_arena]
    function code:check_winner unless @a[tag=winner]
    # 1.13+ syntax
    execute if entity @a[tag=in_arena] run function code:arena_events
    execute unless entity @a[tag=winner] run function code:check_winner
    # 1.13+ syntax (block condition)
    execute if block 0 60 0 air run function code:missing_block
    execute unless block 0 90 0 grass_block run function code:place_block

In 1.13+ you can check more things apart from selectors with [`execute if/unless`](https://minecraft.wiki/w/Commands/execute#Condition_subcommands)

## Slow down your commands

Not everything needs to be running at 20Hz. If you can run some commands at 10Hz instead, you've halved the impact those commands were having. In Bedrock you can just add a delay to a repeating commandblock into the block directly, in Java you need some workaround.

### Success Count
An easy trick with command blocks to make a clock run at half its speed is the following command:

    # pre-1.13 syntax
    testforblock ~ ~ ~ repeating_command_block * {SuccessCount:0}
    # 1.13+ syntax
    execute if block ~ ~ ~ repeating_command_block{SuccessCount:0}
    # 1.13+ syntax (only run one command)
    execute if block ~ ~ ~ repeating_command_block{SuccessCount:0} run <command>

Set up [like this](http://i.imgur.com/OULTCZx.png) (unless running only one command), the command will alternate between succeeding (as it failed last time so has `SuccessCount:0`) and failing (as it succeeded last time so has `SuccessCount:1`), and the conditional repeating block coming off of it will thus activate every other tick.

These cause no block updates and require no entities or scoreboard objectives, but are limited to halving the speed of the first block.


### Command block minecart
The entity [`command_block_minecart`](https://minecraft.wiki/w/Minecart_with_Command_Block) execute the written command every 4 ticks. Keep in mind that people can break the minecart (but it will **not** drop the command block).

### Scoreboard timer
More flexible and commonly used are scoreboard timers. One command continually increments a value, another tests when this value reaches a certain number, then the value is reset and a chain of commands is activated.

    ## pre-1.12
    # in chat
    scoreboard objectives add timer dummy

    # command block
    scoreboard players add FakePlayerA TimerScore 1    
    scoreboard players test FakePlayerA TimerScore 60
    # Chain conditional
    scoreboard players set FakePlayerA TimerScore 0
    # Conditional repeating command
    <any command>

[Here's an example setup image pre-1.13](http://i.imgur.com/fGyA294.png)

    ## 1.13+
    # in chat
    scoreboard objectives add timer dummy

    # command blocks / tick function
    scoreboard players add t.5sec timer 1
    execute if score t.5sec timer matches 100.. run <command/function>
    execute if score t.5sec timer matches 100.. run scoreboard players reset t.5sec timer

These cause no block updates and require no entities, but will require an objective, and a new fake player for each timer. Scoreboard timers can also be used in functions.

### Falling blocks or entities
Other methods such as a falling block clock exist and can be convenient, but cause block updates, lighting updates, and requires an entity.

In this example we use an armor stand with the tag `loop`, we will place a pressure plate with an impulse command block below with the following command:
```
execute as @e[tag=loop,distance=..3,type=armor_stand] at @s run tp @s ~ ~5 ~
```
To add more commands, just add a chain one to the previus impulse.

> [!IMPORTANT]
> It is not recomended to use this method

### Schedule
**Functions** can be even easier to run on slower speeds, as the `schedule` command allows you to run functions after a certain amount of time has passed. You can use this for functions to schedule themselves to essentially create a slower clock.

`code/slow_function.mcfunction`

    schedule function slow_function 10t
    say this will run twice per second

you just need to start the function at some point (e.g. in the `#minecraft:load` function tag) and it will run at a slower speed.

## Avoid chunk updates

Chunk updates occur when blocks change and data is sent to the client. They are one of the major causes of client-side lag. You can see the number of chunk updates in the last second [next to your FPS on the `F3` screen](http://i.imgur.com/G7qvYcF.png). When players aren't loading new chunks and commands aren't performing at world operations, you should aim for this to be 0.

Primarily, turn off TrackOutput on all of your command blocks. This will stop them from causing chunk updates. [Here's an MCEdit filter](https://drive.google.com/open?id=0B5GBricpOPLnLTNSVnpnUE9jXzA) by gentlegiantJGC and updated by ReduxRedstone that should turn off TrackOutput in all of your commands.

`/fill` and `/clone` commands have the potential to cause a large number of chunk updates if used on a clock. Be especially wary near the top of the world, where these commands can cause a cascade of expensive lighting updates. To see just how bad this is, try `/fill ~-15 255 ~-15 ~15 255 ~15 stone` in a world you don't care about, and see how long it takes to complete a single 30×1×30 fill, then imagine that on a clock.

## Avoid redstone

Redstone causes a lot of block and chunk updates, especially redstone wire, and especially if it's on a fast clock. 

Any processing you can do with redstone, you can do with solely commands. Repeaters should no longer be used for delays ([use scoreboard or area effect cloud timers](/wiki/questions#wiki_add_a_delay_to_a_command)), and comparators should no longer be used to check conditions (use conditional blocks, [stats](/wiki/questions/functionconditions), or [tags](/wiki/questions#wiki_activate_a_command_once_when_a_player_does_something_.28e.g.3A_enters_an_area.29))

Trying to reduce how often commands are running by switching from a repeating block to a redstone clock (e.g: comparator) can actually worsen the performance. You should use the methods described in the slowing down your commands section above.

## Optimize selectors

Selectors are used a lot in commands. To make sure you're not causing extra work, it's important to understand a bit about how they work:

1. Fetch a list of possible entities:
    * If the selector type is `@a`, `@p`, or `@r` without a `type`, fetch the list of all logged-in players
    * Otherwise (`@e`, or `@r` with a `type`):
        * If `dx`/`dy`/`dz` or `r` are set, fetch a list of all entities in the chunks covered by that area
        * If none of them are set, fetch a list of all loaded entities
2. Narrow down the list by checking arguments in the following order:
    * `dx`/`dy`/`dz`
    * `type`
    * `level` (`l`/`lm`)
    * `gamemode` (`m`)
    * `team`
    * `scores`
    * `name`
    * `tag`
    * `distance` (`r`/`rm`)  
    * `x_rot`/`y_rot` (`rx`/`rxm`/`ry`/`rym`)
    * `nbt`
3. Sort the remaining list by distance (`@p`, `@a`, `@e` if `sort=nearest` or `sort=furthest`) or randomly shuffle it (`@r` / `sort=random`)
4. Truncate the list based on `limit` / `c`

Your main aim here is to narrow down the list as much as possible before getting to the expensive distance-sorting/shuffling and especially the super expensive NBT checks. You shouldn't however unnecessarily check arguments that won't narrow down the list before distance-sorting/shuffling. For example, don't check `@e[type=zombie,tag=IsUndeadMob]` if all zombies will have that tag. You shouldn't make your selector imprecise so that it applies to more entities than it needs to, either. 

`nbt` is the least efficient target selector, as it has to do a lot of converting and comparing, and as such should be avoided at (almost) all costs, or at least limited to the absolute minimum required. [Predicates](https://minecraft.wiki/w/Predicate), in the other hand, can archive the same causing less performance impact only when not using NBT checks but built-in checks. If using NBT checks it actually performs worse. 

`@s` is the most efficient selector, directly grabbing the command sender. See the section on **function-specific optimisations** for how you can make use of it to cut down on use of other selectors.

As an example of how to optimize selectors: before `@s` existed, if you wanted an entity to select itself, it was better to use `@e[c=1,r=0]` than `@e[c=1]`. This is because the latter will have to distance-sort/shuffle every loaded entity, whereas the former will narrow down the list to just things on the exact same spot before the distance-sorting/shuffling applies.

Be aware that some selector arguments, such as large values of `dx`/`dy`/`dz` (which will cycle through every chunk covered by their region), can be intensive to check.

Consider also whether you actually need a selector. If you're selecting the same one entity that always exists, you could get its UUID and target it with that instead.

## Optimize entities

Using armor stands as markers? Strongly consider switching to area effect clouds or, even better, the marker entity instead. [Here's a good video showing just how much difference this makes.](https://www.youtube.com/watch?v=RKXzWGQfIcg) You can summon an area effect cloud that acts as a marker with:

    summon area_effect_cloud ~ ~ ~ {Duration:2147483647}

These won't show up to spectators, which is a bonus if you don't want spectators to see your markers. If you want to see them for debugging purposes, turn on hitboxes (`F3 + B`). Armor stands should only be used where necessary, such as displaying an item (In pre-1.19.4, as you can use item displays in newer versions) or using `Motion`.

[Marker entities](https://minecraft.wiki/wiki/Marker) on the other hand are impossible to see, as they aren't even sent to the client (unless using [a mod](https://modrinth.com/mod/visiblebarriers), that needs to be on the server as well), so you'd need to resort to other options to debug them. This however means that they have a competitive advantage when it comes to performance and should be used as a marker over the other options wherever possible.  
Additionally they can store any kind of NBT data in their `data` NBT component (though storing abritrary data is probably better stored in the `storage` anyways).

Remember from the previous section that selectors work by getting a list of all loaded entities, then narrowing that down. This means that extra entities increase the workload of every selector in your commands. In some cases, you could be able to use static coordinates rather than executing off of a marker entity. 

Tools like [CommandStudio](http://commandstudio.github.io/commandstudio/) will let you reference back to previous command blocks and will automatically figure out the coordinate offset when compiled, so you don't have to worry about changing the coordinates when you add more commands between them.

## Function-specific optimisations

With functions, rather than running many commands on the same entities (e.g: `@e[tag=blah]`) by repeatedly evaluating `@e[tag=blah]`, like this:

    effect give @e[tag=blah] nausea
    execute at @e[tag=blah] run setblock ~ ~1 ~ stone
    execute at @e[tag=blah] run setblock ~ ~2 ~ grass
    tp @e[tag=blah] ~ ~5 ~
    ...

You can instead put all the commands into a function targeting `@s`, then run that function from those entities:

    execute as @e[tag=blah] at @s run function code:blah

`code/blah.mcfunction`

    effect give @s nausea
    setblock ~ ~1 ~ stone
    setblock ~ ~2 ~ grass
    tp @s ~ ~5 ~
    ...

This means that instead of evaluating `@e[tag=blah]` many times, it is only evaluated once and thus grants a huge performance boost.

## More in-depth optimization

[/u/Wooden_chest](https://www.reddit.com/user/Wooden_chest/) created an in-depth analysis of a lot of small things that can improve your performance, like what order to put your execute subcommands into or that depending on the circumstance it might actually be faster to copy NBT you want to test to the storage first before testing it instead of testing it on the entity/player directly.

Read their full post here: https://www.reddit.com/r/MinecraftCommands/comments/w4vjs3/whenever_i_create_datapacks_i_sometimes_do/
