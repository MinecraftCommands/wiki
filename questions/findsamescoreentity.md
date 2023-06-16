# Find an entity with the same score as another entity or player

related: [Check if a score is equal to, greater than, or less than another score](/questions/scorecompare)

## Java

_For a 1.12 and below solution, have a look at the bedrock solution below._

### Method 1: Compare directly

For this method to work, we need to make sure that the entity that we want to compare to has to be unique in some way, so it can be found and is the only possible entity to result from its target selector. In this case we assume that it has been tagged with the `compare` tag and is the only entity to have that. Furthermore, the objective we're comparing is called `points`.

    execute as @a if score @s points = @e[tag=compare,limit=1] points run I have the same points

This method works by changing the executor to all entities that need to be checked (in this case all players, `@a`) and then using the `execute if score` subcommand to check every executors score against the entities score.

### Method 2: Store the score in a fake player first.

This method has the advantage over the first one that it only need to evaluate the selector for the special entity once, but it has the disadvantage that it needs 2 commands. ([what is a fake player?](/questions/fakeplayer))

    scoreboard players operation #fakeplayer points = @e[tag=compare,limit=1] points
    execute as @a if score @s points = #fakeplayer points run I have the same points

### Method 3: keeping the context through location

In this method we change the execution _location_ to be the player(s) and the execution _entity_ to be the entities to be compared to. That way `@p` is all the players respectively and `@s` is the entity to compare to. We can then get the player back into the execution chain using `@p`. 

    execute at @a as @e if score @s id = @p id run ...

## Bedrock

In bedrock this whole endeavour requires a few more commands, as execute doesn't have any subcommands like that and `/scoreboard players test` only allows for hardcoded ranges. Instead the way to go here is to remove the score from all the entities that need to be checked and then checking whether their score is 0.

Again, we're assuming that the scoreboard objective you want to compare is called `points`, that we want to find any player with the same score, and that the entity to compare to is the only entity tagged with `compare`.

While we could just do the math operations on the original scores, we need to make sure to reset those afterwards if we want to keep the original scores around for one reason or another, so we're going to use a temporary scoreboard objective that we store the value in and can modify it without affecting the original score. This temporary objective is assumed to be called `pointsTmp`.

    # first we copy the score to the new objective
    execute @a ~~~ scoreboard players operation @s pointsTmp = @s points
    # next we deduct the score of the special entity from everyones copy
    execute @a ~~~ scoreboard players operation @s pointsTmp -= @e[tag=compare,c=1] points
    # now we know that anyone with a copied score of 0 has the same score as the entity
    execute @a[scores={pointsTmp=0}] ~~~ say I have the same score as the entity