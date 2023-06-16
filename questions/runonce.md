# Activate a command *once* when a player does something (e.g: enters an area)

This makes a command act as if it was on a comparator, without the lag and multiplayer incompatibility that comes from using a comparator. The general idea here is to select players that match a selector, but did **not** match that same selector in the previous tick. For example, players who have just entered an area (with `@a[x=73,y=10,z=3,distance=..1]`), just gained level 5 (with `@a[level=5]`), just entered creative (with `@a[gamemode=creative]`), etc.  

The following commands, running in this order, will keep track of whether a player matched the selector `@a[x=73,y=10,z=3,distance=..1]`:

    tag @a[tag=alreadyMatched] remove alreadyMatched
    tag @a[x=73,y=10,z=3,distance=..1] add alreadyMatched

Players who matched the selector will get the `alreadyMatched` scoreboard tag (you can call this scoreboard tag whatever you want, so long as you're consistent with it). At the start of the next tick, players who matched the selector in the previous tick will have the `alreadyMatched` scoreboard tag. This means that we can select players who don't have the scoreboard tag (`tag=!alreadyMatched`, meaning they didn't match in the previous tick) but do *now* match the selector:

    execute as @a[x=73,y=10,z=3,distance=..1,tag=!alreadyMatched] run say I just entered the area!
    tag @a[tag=alreadyMatched] remove alreadyMatched
    tag @a[x=73,y=10,z=3,distance=..1] add alreadyMatched