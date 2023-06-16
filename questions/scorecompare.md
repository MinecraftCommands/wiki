# Check if a score is equal to, greater than, or less than, another score

## 1.13 and above

In 1.13, the `if score` execute subcommand makes this a lot easier. The syntax you'll want is:

    execute if score <target> <targetObjective> (<|<=|=|>|>=) <source> <sourceObjective> run <command>

For example, to check if the executer's `kills` is greater than their `deaths`:

    execute if score @s kills > @s deaths run say I have more kills than deaths!

Or, to check if the nearest player's `money` score is equal to their `cost`:

    execute if score @p money = @p cost run <command>

You can also use `matches` to check for a range if you want to always check for a given range and don't want to store that in a different scoreboard:  

## 1.12 and below

To do this we must take one score from another, check if the score is now equal to/greater than/less than 0, then add the score back (to restore the first score's original value).

For example, select all players whose `kills` score is greater than their `deaths` score:

1. Take everyone's `deaths` from their `kills`

        execute @a ~ ~ ~ scoreboard players operation @s kills -= @s deaths

2. Select all players that now have a positive `kills` score (so they had more kills than deaths)

        say @a[score_kills_min=1]

3. Add everyone's `deaths` back to their `kills`, to restore `kills` original value

        execute @a ~ ~ ~ scoreboard players operation @s kills += @s deaths

        execute if score @p money matches 10.. run say I have 10 money or more.