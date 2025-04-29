# Creating a circle with commands

Creating perfect (or as perfect as they can be with minecrafts blocky nature) circles by hand can be annoying, daunting even. And while there are [tools that help you lay out block circles](https://minecraftcirclegenerator.co/), it might be easier to write a few commands to do it for you.

## The idea

The idea of how to achieve this is pretty simple: Have an entity in the center that slowly rotates, then place what you want X blocks in front of it using the local coordinates (`^ ^ ^X`).

## Implementation

For the example we're assuming that we want to make a block outline. You can easily adjust this to summon entities or make a filled circle by swapping the `/setblock` command for something else (e.g., `/summon`, `/ fill`).

1. Summon your center entity. We're using an armorstand for parity, but in Java you can use other things like a [`marker`](https://minecraft.wiki/w/Marker) (for better performance) or any NoAI entity.

```mcfunction
/summon armor_stand ~ ~ ~
```

2. tag the armorstand so we can reference it easily.

```mcfunction
# Bedrock
/tag @e[type=armor_stand,c=1] add center
# Java 1.13+
/tag @e[type=armor_stand,sort=nearest,limit=1] add center
# Java 1.21+
/tag @n[type=armor_stand] add center
```

3. now, to automatically place the blocks, we'll want to put the following command into a repeating commandblock and the ones after that into chain commandblocks attached to the repeating one. Starting off with placing the block in front of the armorstand.

        execute as @e[tag=center] at @s run setblock ^ ^ ^10 stone

4. Next, we rotate by what seems a good rotation to cover the entire circle. You can do some math to figure out a good stepsize, or you can just decrease the size if you end up with holes in your circle.

    The math works like this: You calculate the radian distance as follows: `d = (deg / 360) * 2 * π * radius`. Or, if we solve for the degrees needed from a distance between blocks: `deg = (180 * d) / (π * radius)`. So if your radius is 10 blocks as in the example above, and you want to place a block every one block of distance, you should rotate by (180 * 1) / (π * 10) = 5.7°. But with minecrafts blocky nature, if we want to place continuous blocks, we might just want to aim for half of that.  

        execute as @e[tag=center] at @s run tp @s ~ ~ ~ ~ ~2.35

5. Power the repeating commandblock until you have the circle you desire.

You can of course expand on this basic system to make it more modular, run it exactly as many times as is needed to complete the circle (especially important for summoning entities), etc. 

## Potential issues

If you are trying to use this to make really big circles (hundreds of blocks in diameter), you might find that this won't work as you intend it to, seemingly skipping many steps in between.  

We're unsure as to why exactly this happens, but our best guess is that minecrafts internal calculations are limited to a certain rotational precision (e.g., angles get rounded to 0.01° for calculations). This doesn't have any noticeable effect on 99.99% of gameplay, but becomes visible in these situations.