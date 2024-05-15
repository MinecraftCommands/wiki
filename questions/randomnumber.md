# Generate a random number

This article talks about the Java Edition of the game. In Bedrock, you can just use `scoreboard players random` to get a random number into your scoreboard and don't need to go through all this hassle.

> [!NOTE]
> **None of these numbers are truly random**. They are all only "pseudo" random (which means they only feel like they are random to a human, but are using some form of deterministic algorithm behind the scenes) because that's how computers work. They will be refered to "random" generator for simplicity. This likely won't affect your contraption, but it's important to point out.

## /random command
**This method is, currently, the best, there is no reason to use the others**

Effective Range [-2'147'483'648 to 2'147'483'647]

This command is used to generate a random number specifying the maximum and the minimum, this is the best method of all because it is the easiest to make. More information can be found on the wiki.

First we need to create a scoreboard where we will store the random number.

    /scoreboard objectives add random dummy
We need to store the result of the /random command to a fake player

    /execute store result score <player/fakeplayer> <scoreboard> run random value <min>..<max>
    
For example:
execute store result score #command random run random value 1..5
And now we need to check the value of the scoreboard, in this case we used numbers from 1 to 5 so we use one command to check every possible scoreboard value.

    execute if score #command random matches 1 run <command 1>
    execute if score #command random matches 2 run <command 2>
    execute if score #command random matches 3 run <command 3>
    execute if score #command random matches 4 run <command 4>
    execute if score #command random matches 5 run <command 5>

Or we can use [ranges](wiki/questions/ranges) to detect more of one number.

    execute if score #command random matches 1..3 run say 1, 2 or 3
    execute if score #command random matches 4..5 run say 4 or 4

## without /random command

There are many ways to get a random number in minecraft. the first two are arguably the best as they have the least limitations and require the least work to set up.

## Have something happen with a random chance

For this special case, you don't need any of the random number methods, thanks to 1.15's predicates. A simple predicate can be used to run something with an e.g. 1/10 chance like this:

    {
        "condition": "minecraft:random_chance",
        "chance": 0.1
    }

If you need to choose one of 10 things at random, you should read on.

## 1: PRNG/LCG

Effective Range: Depends on the implementation, one of the ones on the discord server has [0, 130000]

You can create a PRNG (pseudo random number generator, pseudo because it's not really random, but to a human it looks random enough) using just a few commands that do some math. Check out the [wikipedia about PRNGs](https://en.wikipedia.org/wiki/Pseudorandom_number_generator). The most common one and easiest to implement is probably the [linear congruential generator](https://en.wikipedia.org/wiki/Linear_congruential_generator).

If you don't want to do it yourself, you can go on the [discord](https://discord.gg/9wNcfsH) and check the #resources channel, there are a few PRNGs you can download and use in there. Alternatively you can [check out this post](https://www.reddit.com/r/MinecraftCommands/comments/vv68n6/tutorial_random_number_generator_by_scoreboard) by /u/mingshi3_uiuc for an implementation to copy.

_Probably the best listed method if the range is enough for you, as it creates the least amount of server strain while having a very high range that should be enough for most purposes._

## 2: UUID

Effective Range: [-2147483648, 2147483647]

Minecraft already does a bunch of randomization. One of them is generating the UUIDs of entities. This means that you can summon a new entity, `execute store ... data get` to get the UUID and then use scoreboard operations to shrink it into your desired range. ***Note that even though the UUID value being returned is within the full integer range, the `%=` scoreboard operation changes it into a value that's `0` or greater. In versions before 1.13.1, this would also give you negative values.***

Let's assume there are two dummy scoreboards called `random` and `range` and we want a random score on the executing entity's `random` scoreboard, in the range of the entity's `range` scoreboard.  
Using the UUID method it could look like this:

	summon area_effect_cloud ~ ~ ~ {Tags:["random_uuid"]}
	execute store result score @s random run data get entity @e[type=area_effect_cloud,tag=random_uuid,limit=1] UUID[0] 1
	scoreboard players operation @s random %= @s range
	kill @e[type=area_effect_cloud,tag=random_uuid]

_The second best option listed here. More taxing on the server due to it's creation and removal of entities as well as NBT access, so better limit its use as much as possible. Use only if Method 1 doesn't work for you or if you need a specifc player to always get the same random number (their uuid doesn't change)._

***<1.16*** In versions before 1.16, UUIDs were split into two separate NBT tags called `UUIDLeast` and `UUIDMost`. To make the example above compatible, you'll need to replace the second command with this:

    execute store result score @s random run data get entity @e[type=area_effect_cloud,tag=random_uuid,limit=1] UUIDMost 0.00000000023283064365386962890625

The NBT tags `UUIDLeast` and `UUIDMost` are both the size of a `long` (64 bits), while `data get` can only return an `int` (32 bits). To shrink the `long` sized value into the size of an `int`, we use the scale value `0.00000000023283064365386962890625` which we get from the value of `1 / (2^32)`. This effectively gives us the upper half of the `long`.

## 3: Loot Tables

Effective Range: [0, 10000] (in theory all the way up to 2^(31)-1, but since you're summoning an entity which gets deleted instantly again for this to work, anything above a few thousand will likely lag your game).

This method takes advantage of the fact that using `execute store` with the `loot spawn` command will return the amount of rolls the loot table did, because it summons that many item entities. So if we use a loot table that creates a 0 amount item, we can safely use it to get our random number without creating any items in the world. _The item entity is still created but immediately removed again! This is what limits the range of this method._

Example loot table:

    {
      "type": "minecraft:empty",
      "pools": [
        {
          "rolls": {
            "min": 1,
            "max": 5
          },
          "entries": [
            {
              "type": "minecraft:item",
              "name": "stone",
              "functions": [
                {
                  "function": "minecraft:set_count",
                  "count": 0
                }
              ]
            }
          ]
        }
      ]
    }

Change the rolls to fit your desired range (not negative, 0 is allowed), and/or use the method described below to fit it into a range using modulo, so you don't need a different loottable for every different range.

Command to get the value from the table:

    execute store result score @s <OBJ> run loot spawn ~ ~ ~ loot example:random_loot_table

_The third best option, with a small effective range. Use only for smaller amounts of values and if methods 1 and 2 don't work for you._

_Taken from [this](https://www.reddit.com/r/MinecraftCommands/comments/evoghb/another_rng_method/) post on the subreddit_.  

## 4: item rotation

**Outdated**

Effective Range: [0, 360]

A way to get a random number before 1.13 in the range of up to 360 numbers was to summon an entity like a squid, kill it and check the rotation of the inksac item entity it just dropped, since that rotation is always random. It's very much deprecated now though, thanks to Number 2 in this list.

## 5: @r / @e[sort=random]

Effective Range: [0, 10] (theoretically infinite, but practically anything above 10 is not worth it)

Before we got the (arguably much better) solutions above, we could use `@e[limit=1,sort=random,tag=randomizer]` to select a random entity, which we would've each given their respective scoreboard score (or put the commandblock to be triggered below them or something similar). This has the obvious disadvantage, that you'll need one entity per possible score. as long as it's below 10 this is still doable, anything above does get tedious. To avoid performance issues, use a [marker entity](https://minecraft.wiki/w/Marker)

## 6: spreadplayers

**Outdated.**

Effective Range: [0, 100] (theoretically 900000000000000 if you're using every single block in a minecraft world as a position, practically even 100 is a stretch)

Another Minecraft-included way for randomisation is `/spreadplayers`, which will position an entity randomly over a predefined area. Depending on where it lands you could have a commandblock below triggered for it to then run the desired commands. Seems to have a bias agains the edges of the area, so make the spreadplayers area in the command bigger than the actual possible spawning spaces.   

A different way to use the spreadplayers randomisation is to `data get` the position the entity ends up on. This only works reliably in loaded chunks. Since spreadplayers does load the chunk the entity is placed in briefly, it _may_ work when using a function or chained commands, but this method of chunkloading has been quite unreliable in the past.

## 7: running score

A very primitive way to get somewhat random numbers but probably the easiest one. It relies on time to get a random number, so this is not applicable for when random numbers are needed in predefined time lengths (X amount of ticks, maybe even the same tick) and **it is advised not to use this**, as there are much better alternatives. How it works is you basically count up a scoreboard from the minimum to the maximum random number, incrementing by one per tick and resetting it to the minimum once you reached the maximum. Then to get a "random" number you just take whatever number this running score happens to be on at that moment.

## Get the number into the desired range

Many of the generators above have a range that only covers positive numbers, reaching from 0 up to X. But what if you need a random number between 5 and 15? what about -10 and 10?  

The answer is very simple: you just shift the result up or down by applying an addition or substraction.   

`max-min = needed range`, so to get from the result to the desired range we can simply do `result + min = needed range`.

Let's suppose for an example that we're trying to get a score between min=-10 and max=10. That means we have a needed range of `10 - (-10) = [0,20]`. So we set up our RNG to create a number between 0 and 20.  
Then we shift the result down by 10 by adding the -10 (or substracting the 10): `[0,20] + (-10) = [-10,10]`.

Another example on how to get numbers between 5 and 15:  
needed: `15-5=[0,10]`  
desired: `[0,10] + 5 = [5,15]`
