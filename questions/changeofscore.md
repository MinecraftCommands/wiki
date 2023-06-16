# Detect a change of score

_Both Java and Bedrock. Java Syntax used if not stated otherwise._  

If you need to detect when and if a score of a player has changed, there are two ways to go about this. This is generally most useful if you're using a scoreboard objective that changes because of something the player is doing (e.g. clicking, attacking, sneaking) or the game is doing to the player (e.g. taking damage, change of health and hunger).

If you are changing the score yourself, it is generally advised to also do the effects to the player yourself, as you're changing the score anyways (which is currently the case in 100% of the time in Bedrock), but in some edge cases this can still be useful to do anyways.

## The destructive method

This is the more commonly used method, because in many use cases it's not important to know the total score, just that the score changed. It is generally used for [item click detection](/wiki/questionss/itemclick), [player deaths](/wiki/questionss/playerdeaths) in minigames and more and involves resetting the score back to 0 once it has been changed.  
_Some objective types are read-only, such as Health or Hunger. Those won't work for this method._

So lets assume you have a score that detects a players death and want to do something with that player everytime they die once.

    execute as @a[scores={deaths=1..}] run say I died :(
    scoreboard players set @a[scores={dead=1..}] deaths 0

You can either `set` the score back to 0, or `reset` the score, whichever one fits your system better. In most cases either one will work fine.

## The non-destructive method

Sometimes you have an objective you don't want to or cannot change. In those cases it's more difficult to know when an objective changed from the last tick to the current one and involves a second, type dummy objective which we use to store the previous value to be able to compare them. Assuming you're running this as the entity you want to check, have the objective called `deaths` and the additional one called `tmp`, the commands could look like this:

    execute unless score @s tmp = @s deaths run say My score has changed in the last tick!
    scoreboard players operation @s tmp = @s deaths

In Bedrock we lack the `if/unless score` argument, so we'll need to do a slightly different and longer approach:
    
    scoreboard players operation @s tmp -= @s deaths
    tag @s[scores={tmp=0}] add noChange
    execute @s[tag=!noChange] ~ ~ ~ say My score has changed in the last tick!
    tag @s remove noChange
    scoreboard players operation @s tmp = @s deaths