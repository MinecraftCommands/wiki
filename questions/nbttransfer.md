# Store an NBT value to a score, and vice versa?

Any **numeric** NBT tag, even inside lists, can be set to, or gotten into, a scoreboard value.

## NBT --> Score

The `/data get` command allows you to specify a block or entity, and retrieve NBT data from it. For example:

```py
data get entity @s
```

Rather than printing out all data, you can specify a "path" to get one specific tag, like so:

```py
data get entity @s Pos[0]
```

Pos[0] means the first (indexes counting from 0) element of the `Pos` list, so the x coordinate (you can access tags inside of compounds as `compound.tag`).

This is cast into an integer, meaning if you are at `x=73.1031`, the command's result will be just `73`.   
The optional `[<scale>]` factor lets you multiply the number by something before it's read to an integer, for more accuracy. For example, the following would have the result `7310` (100*x):

```py
data get entity @s Pos[0] 100
```

Next we need to store the value of `/data get`'s result into a scoreboard objective, which is exactly what `/execute store` allows us to do:

```py
execute store result score <target> <objective> run <command>
```

This will run `<command>`, and store the result into the the `<target>`'s `<objective>` score. So, putting it together, you could do something like this:

```py
execute store result score @s y_pos run data get entity @s Pos[1] 100
```

Which will store 100 times the executer's y position into their `y_pos` score.

## Score --> NBT

Using NBT paths mentioned in the previous section, `/execute store` will also allow us to store the result of a command into any numeric NBT tag (other than players, you still can't edit player data). The syntax is:

```py
execute store result entity <target> <path> (byte|double|float|int|long|short) <scale> run <command>
execute store result block <pos> <path> (byte|double|float|int|long|short) <scale> run <command>
```

The `<command>` you'll want to run is `/scoreboard players get <target> <objective>`, which returns a specified score as its result.

`<scale>` is similar to how it was for `/data get`; the result will be multiplied by this then written to the NBT path. If you scaled up a position 100* when getting it into a score to preserve accuracy, now is the point you'll want to scale it back down (with a scale of 0.01).

`(byte|double|float|int|long|short)` is the **data type** to cast this value to, as NBT has multiple ways to represent a number (whereas scores are always `int`s). You'll need this to match the data type of the tag you're storing the value into. 

As an example, setting the nearest creeper's y-motion to 0.05 times the executer's `y_speed` score:

```py
execute store result entity @e[type=creeper,limit=1,sort=nearest] Motion[1] double 0.05 run scoreboard players get @s y_speed
```

### NBT --> NBT

You can even store directly from an NBT path to another NBT path. Keep in mind though that command results are always cast to integers, even when you just want to transfer from a float to a float.

For example, to have the nearest pig copy the nearest chicken's x-pos:

```py
execute store result entity @e[type=pig,limit=1,sort=nearest] Pos[0] double 0.01 run data get entity @e[type=chicken,limit=1,sort=nearest] Pos[0] 100
```