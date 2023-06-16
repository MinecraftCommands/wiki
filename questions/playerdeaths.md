# Detect Player Death

_Related: [Detect Player Kills](/wiki/questionss/playerkills)_

## Java

In Java, detecting a dead player is relatively easy. You just set up a scoreboard objective of type `minecraft.custom:minecraft.deaths` and whenever that one increases, the player just died (and is likely still dead).

Likewise, using `minecraft.custom:minecraft.time_since_death` you can detect a player who just respawned, since this objective will stay 0 during the death screen and start counting up the moment the player clicks "respawn".

## Bedrock

In Bedrock the same question is more difficult to answer, since at the time of writing only the `dummy` objective type exists.

The best way to do this in bedrock is with the following commands, in this order:

    /tag @a add dead
    /tag @e[type=player] remove dead
    /scoreboard players add @a[tag=dead,tag=!still_dead] deathCount 1
    /tag @a add still_dead
    /tag @e[type=player] remove still_dead

Set up a dummy scoreboard called `deathCount` and it will count up every time a player dies.  

This works because the `@a` selector selects all players, but the `@e` selector can only select living entities.

_This system was first suggested on the subreddit by /u/Sprunkles137 [here](https://old.reddit.com/r/MinecraftCommands/comments/g5b4n8/challenge_1/fo3p5p0/)._

There is a different way as well, which can only be used in a very controlled environment where players cannot set their own spawnpoints with beds but all spawning is controlled by the mapmaker. This way you can set everyone's spawnpoint to an otherwise inaccessible position in the world, then detect players respawning there, count up their death score and teleport them to the "actual" spawnpoint. **It is not advised to use this system anymore, as the first system is easier to do, requires less setup and has a wider range of applicable possibilities!**