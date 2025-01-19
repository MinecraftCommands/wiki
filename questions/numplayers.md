# Check if there are exactly X players (or entities) matching a selector?

For example, you want to check if a team has exactly 2 players. 

| 📝 Note |
|---------|
|if a selector such as `@a[team=red]` selects `Alice, Bob, Carol, Dave`, then `@a[team=red,limit=2]` will select `Alice, Bob` and run the command on those players. `limit=2` does not mean *"only run if there are 2 players"*, but rather *"limit the players selected to at most 2"*. (Same for `c` in bedrock).|

A command will still run (and potentially succeed once) even if `@a[team=red,limit=2]` only finds one player, and succeeding once is all that is needed for conditional blocks coming off of the command block to run.

## Java and Bedrock
| 📝 Note |
|---------|
|This method will work in Java too, but it's recommended to use the Java exclusive method|

The easiest way to do this in bedrock is to have the found entities count up the score of another entity or a [fake player](/wiki/questions/fakeplayer) (the latter is currently a little harder to check for their score, so it is not used in this tutorial).

Using a dummy scoreboard objective named `result` and an entity with the `counter` tag, we can count how many players there are in team red:

    scoreboard players set @e[tag=counter] result 0
    execute as @a[team=red] run scoreboard players add @e[tag=counter] result 1
    execute as @e[tag=counter,scores={result=2}] run say there are exactly 2 people on the red team

You can also use [ranges](wiki/questions/ranges)

    execute as @e[tag=counter,scores={result=2..}] run say there are 2 or more people on the red team
    execute as @e[tag=counter,scores={result=..2}] run say there are 2 or less people on the red team
    execute as @e[tag=counter,scores={result=2..9}] run say there are between 2 and 9 people on the red team

## Bedrock
| 📝 Note |
|---------|
|It is not recommended to use this method|

You can use a comparator with the `/testfor` command, the output, in signal strength, is the same as the number of online players.

## Java 

### Method 1: Storing the found entities using `execute store`

Introduced in 1.13 and replacing the `/stats` command, the `store` subcommand of `execute` is able to store the result of the command after it in multiple ways, including into a scoreboard.  
Using a dummy scoreboard objective name `result` and the [fake player](/wiki/questions/fakeplayer) `#count`, we can count the amount of entities a command finds and then execute off of that. Note that we can use [ranges](wiki/questions/ranges)

    execute store result score #count result if entity @a[team=red]
    execute if score #count result matches 2 run say there are exactly 2 players on team red.

The fake player can be replaced by an entity, although it is not recommended, which then changes the second command to  

    execute if entity @e[<your entity>,scores={result=2}] run say there are exactly 2 players on team red.

It also works with [ranges](wiki/questions/ranges)

    execute if score #count result matches 2.. run say there are 2 or more players on team red.
    execute if score #count result matches ..2 run say there are 2 or less players on team red.

### Method 2: Checking the command block's `SuccessCount`

| 📝 Note |
|---------|
|This method is outdated and should not be used in current versions of the game|
|For the sake of allowing users of 1.12 and below to access this for them still useful information, it is kept around|

When a `/testfor` command is run, the command block's `SuccessCount` NBT tag will be set equal to the number of entities that the selector found. You can then use a `/testforblock` command in another block to check whether the first command block's `SuccessCount` is a certain number.

For example, this will test for *exactly* 2 players on the red team (no more, no less):

    testfor @a[team=red]
    testforblock <x> <y> <z> command_block -1 {SuccessCount:2}

Change `<x> <y> <z>` in the second command block to the coordinates of the first command block. You may also need to change `command_block` to `chain_command_block` or `repeating_command_block`.

The second command will only succeed if the first command has a SuccessCount of 1 (it found exactly 1 player). You can then run a conditional chain block off of that second command to activate whatever you want to happen when there is exactly 1 player:

### Method 3: With `/stats` (1.8-1.12)

| 📝 Note |
|---------|
|This method is outdated and can not be used in current versions of the game|

Stats are harder to understand and set up, but gives you more flexibility (works in functions, can test a range rather than exact values, can test things other than SuccessCount). If you have not used `/stats` in the past, you will need to watch/read a tutorial (or multiple) and play around with them until you are confident in its usage. Then:

 * Set an entity's `SuccessCount` to store in a scoreboard objective (e.g: its `SuccessScore`)
 * Execute off of that entity to run the command (e.g: `testfor @a[team=red]`)
 * You can now check `score_SuccessScore_min=2,score_SuccessScore=2` of that entity
