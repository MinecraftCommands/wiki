# Detect a player joining

* [Java](#java)
* [Bedrock](#bedrock)

For this, we'll have to differentiate between players joining for the first time and players *re*joining the world for the Nth time, as the approaches are slightly different.

| 📝 Note |
|---------|
|In this example we are using a tellraw command, but in 1.21.5+ the tellraw syntax has changed, so you will need to update it accordingly.|

## First time
A player that joins a world for the first time is a blank slate, they have no scores, no tags, no advancements, no nothing on them. We can use this to detect the lack of an initialization tag on the player, apply all our actions to them, then give them the tag.

### Tags

execute as @a[tag=!init] run tellraw @a ["",{"selector":"@s"},{"text":" just logged in for the first time!"}]
tag @a[tag=!init] add init

### Advancements
Another way to do this is using an advancement in a datapack, this advancement will we granted every tick, so new players (that don't have this advancement) will receive it and will run the function specified in the reward.

```py
# advancement example:first_join
{
 "criteria": {
   "requirement": {
     "trigger": "minecraft:tick"
   }
 },
 "rewards": {
   "function": "example:first_join"
 }
}

# function example:first_join
tellraw @a [{"selector":"@s"}," just logged in for the first time!"]
```

## Consecutive Time

### Java 
 
You can set up an objective of type `minecraft.custom:minecraft.leave_game`, which will count up the moment a player leaves the server, which you can then detect the moment they come back, because, even if the score has been updated, the player is not online and we can't target it with `@a[scores={leave=1}]`.
Assuming you called the objective `leave`, it could look like this:  

```py
execute as @a[scores={leave=1..}] run tellraw @a ["",{"selector":"@s"},{"text":" just came back to us!"}]
scoreboard players reset @a[scores={leave=1..}] leave
```

| 📝 Note |
|---------|
|This method counts exits from the game, but not joins to the server, because the score increases when the player leaves, but since you cannot target an offline player using the target selector, we cannot check that the score has been changed and this can only be done when the player joins the server again|

### Bedrock

In Bedrock we don't have the luxury of the `leave_game` objective, so we'll need to find a workaround. One such workaround could be to have a fake player count up a score every tick/second, then check whether the player has the same score, if not run whatever you want on them, then set the score to the same score as the fake player. Example with a dummy scoreboard objective called "online":

```py
scoreboard players add @a online 1
scoreboard players add .total online 1
scoreboard players operation @a online -= .total online
execute as @a[scores={online=..-1}] run say I relogged!
scoreboard players operation @a online = .total online
```

This seems to be working even in singleplayer.   
However, be aware that it will stop working after 3 years of the world being active if you're running it every tick due to integer overflow. If you change it to run every second instead, this system will last you 60 years.

## Both

### Java
We can combine these two if you want the same thing to happen in both cases. The easiest here would of course be a function that's just run in both cases, but it'd be similarly easy to remove the tag from anyone with the score so you don't need to do everything twice but instead just need one additional command:

```py
tag @a[scores={leave=1..}] remove init
execute as @a[tag=!init] run say I relogged!
tag @a[tag=!init] add init
scoreboard players set @a[scores={leave=1..}] leave 0
```

But it can be simplified to only 2 commands (and one for creating the scoreboard).
First we detect if the player has no set score (so they are a new player) and we store the success of the tellraw. The score is now set to 1 (from no score at all), because the tellraw always succeeds.
Then if the value is not 1 (so it is 2, so they leaved the game) we will store the succes of another tellraw command in the scoreboard, because it is the success it will set it to 1.

```py
# In chat
scoreboard objectives add leave custom:leave_game

# command blocks
execute as @a unless score @s leave = @s leave store success score @s leave run tellraw @a [{"selector":"@s"}," just logged in for the first time!"]
execute as @a unless score @s leave matches 1 store success score @s leave run tellraw @a [{"selector":"@s"}," just came back to us!"]
```

### Bedrock

In bedrock we don't have `execute store` so we will need to split the command in 2. 
| 📝 Note |
|---------|
|This method is the same as the 2 others merged, nothing else changed|

```py
# In chat
scoreboard objectives add online dummy

# Command block
scoreboard players add @a online 1
scoreboard players add .total online 1
scoreboard players operation @a online -= .total online
execute as @a[scores={online=..-1}] run say I rejoined
scoreboard players operation @a online = .total online
execute as @a[tag=!init] run say I joined for the first time
tag @a[tag=!init] add init
```