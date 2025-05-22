# Target a player above/below a certain Y level

  - [`dy`](#dy)
  - [predicate](#predicate)

## `dy`
For example, you want to teleport players to a certain point if they go above y=200. To do this, we will use the `dx`/`dy`/`dz` cuboid volume selectors, which allow us to specify a width/height/length in which to match a player.

An incorrect first attempt to match players above y=200 might look like:

```mcfunction
say @a[y=200,dy=500]
```

The problem with this is that, as they are not specified but `y` is, `x` and `z` will default to the `x` and `z` of the current executing position, which is in this case the thing running the command. So you'll find that this works if you run it in chat (because it defaults to your `x`/`z`), but won't work when you run it in a command block unless you have the same `x`/`z` as the command block (directly above/below it). Something similar is true for `dx` and `dz`: Because `dy` is specified, the other two default to `0`. Which means the command will internally look like this (assume it's run from a commandblock located at `1 2 3`):
   
```mcfunction
say @a[x=1,y=200,z=3,dx=0,dy=500,dz=0]
```

Instead, we'll need to `execute at` the player, so that the `x` and `z` will default to the players own `x` and `z`, which they will always match:

```mcfunction
execute as @a at @s run say @s[y=200,dy=500]
```

The same technique can be used to find players above/below any `x`/`y`/`z` coordinate.

To detect below a certain height, you can make the `dy` negative.

```mcfunction
/execute as @a at @s run say @s[y=-30,dy=-500]
```
this will detect any players below `-30`

## Predicate

Predicates have a built in check for the `y` values (alongside with the other axis). Keep in mind that this is Java exclusive and will need a datapack if you are in pre-1.20.5.

```json
{
  "condition": "minecraft:entity_properties",
  "entity": "this",
  "predicate": {
    "location": {
      "position": {
        "y": {
          "min": 40
        }
      }
    }
  }
}
```