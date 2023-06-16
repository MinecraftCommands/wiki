# Do something if a player is in certain areas

_Java Syntax, but this can be applied to Bedrock just as well by changing the selector arguments to Bedrock Syntax._

This mostly comes up as a question to change the gamemode in a certain area (e.g. spawn, safe zones, etc), so we will focus on that, but this can be applied to any use case. For questions to do something once a player enters a single area, [look here](/wiki/questionss/runonce).

The naive approach would be to put everyone in a radius X around the worldspawn (e.g.at 0 0 0) to adventure mode and everyone outside of it to survival mode.

    gamemode adventure @a[x=0,y=0,z=0,distance=..X]
    gamemode survival @a[x=0,y=0,z=0,distance=X..]

This method quickly falls apart if you have a non-spherical or non-box shaped area or you want this to apply to more than one area, because a player will always be outside of the other areas, even if they are inside one, so will always end up in survival.

The best way to make sure players are in one of multiple areas without overwriting each other, is to use a tag: So instead of applying the desired effect to each area individually, you tag all players that are in one of the areas and apply the effect once to all of them (or everyone else).

    tag @a remove inArea
    tag @a[x=0,y=0,z=0,distance=..X] add inArea
    tag @a[x=100,y=64,z=100,dx=70,dy=16,dz=28] add in Area
    ....
    gamemode adventure @a[tag=inArea]
    gamemode survival @a[tag=!inArea]

To prevent chatspam for the players and unnecessary gamemode changes, we suggest using the gamemode selector argument on the last two commands:

    gamemode adventure @a[tag=inArea,gamemode=!adventure]
    gamemode survival @a[tag=!inArea,gamemode=!survival]

If for some reason you want to keep commandblockoutput on and don't want your output to be spammed by this system, check out this post by /u/Afanofall23:  
https://www.reddit.com/r/MinecraftCommands/comments/mw11xm/do_something_to_players_in_multiple_specific/