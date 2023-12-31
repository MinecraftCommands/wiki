# Detect a player joining

For this, we'll have to differentiate between players joining for the first time and players _re_joining the world for the Nth time, as the approaches are slightly different.

## First time

A player that joins a world for the first time is a blank slate, they have no scores no tags no nothing on them. We can use this to detect the lack of an initialisation tag on the player, apply all our actions to them, then give them the tag.

    execute as @a[tag=!init] run tellraw @a ["",{"selector":"@s"},{"text":" just logged in for the first time!"}]
    tag @a[tag=!init] add init

## Consecutive Time

### Java 
 
You can set up an objective of type `minecraft.custom:minecraft.leave_game`, which will count up the moment a player leaves the server, which you can then detect the moment they come back. Assuming you called the objective `leave`, it could look like this:  

    execute as @a[scores={leave=1..}] run tellraw @a ["",{"selector":"@s"},{"text":" just came back to us!"}]
    scoreboard players set @a[scores={leave=1..}] leave 0

### Bedrock

In Bedrock we don't have the luxury of the `leave_game` objective, so we'll need to find a workaround. One such workaround could be to have a fake player count up a score every tick/second, then check whether the player has the same score, if not run whatever you want on them, then set the score to the same score as the fake player. Example with a dummy scoreboard objective called "online":

    scoreboard players add @a online 1
    scoreboard players add .total online 1
    scoreboard players operation @a online -= .total online
    execute @a[scores={online=..-1}] ~ ~ ~ say I relogged!
    scoreboard players operation @a online = .total online

This seems to be working even in singleplayer.   
However, be aware that it will stop working after 3 years of the world being active if you're running it every tick due to integer overflow. If you change it to run every second instead, this system will last you 60 years.

## Both

We can combine these two if you want the same thing to happen in both cases. The easiest here would of course be a function that's just run in both cases, but it'd be similarly easy to remove the tag from anyone with the score so you don't need to do everything twice but instead just need one additional command:

    tag @a[scores={leave=1..}] remove init
    execute as @a[tag=!init] ....
    tag @a[tag=!init] add init
    scoreboard players set @a[scores={leave=1..}] leave 0