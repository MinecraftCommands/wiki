# Detect the player moving/sneaking/crawling/sleeping


## Moving
### Predicate
To detect the player moving, you can use the following predicate:

```json
{
  "type": "minecraft:entity_properties",
  "entity": "this",
  "predicate": {
    "movement": {
      "horizontal_speed": {
        "min": 0.01
      }
    }
  }
}
```
For versions before 26.3 the `type` field is renamed to `condition`.

This can be used as a target selector `@e[predicate=...]` or with `execute if predicate`, but then it must be `as` an entity

### Commands
In older versions, where this predicate didn't exist, you can store the position values and compare to the previous tick. Keep in mind that this is performance intensive, and should be minimized, used with a delay or ran only when needed.

```mcfunction
# In chat
scoreboard objectives add Pos.x dummy
scoreboard objectives add Pos.x.copy dummy
scoreboard objectives add Pos.z dummy
scoreboard objectives add Pos.z.copy dummy

# Command blocks
execute as @a run scoreboard players operation @s Pos.x.copy = @s Pos.x
execute as @a run scoreboard players operation @s Pos.z.copy = @s Pos.z
execute as @a store result score Pos.x run data get entity @s Pos[0] 100
execute as @a store result score Pos.z run data get entity @s Pos[2] 100
execute as @a unless score @s Pos.x = @s Pos.x.copy run say I'm moving on the x axis
execute as @a unless score @s Pos.z = @s Pos.z.copy run say I'm moving on the z axis
```

We have 4 scoreboards. `Pos.x` and `Pos.z` will be where the position will be stored and the `.copy` scoreboards contains the value of the previous tick.

First, we copy the values from the previous tick into the copy scoreboards (multiplied by 100 to retain decimal places), then, we update the current values from the data of the player `Pos[0]` and `Pos[2]`.

Last, we compare if the values aren't the same, if they aren't, we know The player moved.

If you want to run a command when the player moves with this method but don't care if it's the X or Z axis then you can change the last 2 commands to add a tag, the run any command as the player with the tag and lastly remove the tag.

```mcfunction
execute as @a unless score @s Pos.x = @s Pos.x.copy run tag @s add moving
execute as @a unless score @s Pos.z = @s Pos.z.copy run tag @s add moving
execute as @a[tag=moving] run say I'm moving
tag @a remove moving
```

## Sneaking

## Predicate

You can use the `flags` predicate to detect sneaking

```json
{
  "type": "minecraft:entity_properties",
  "entity": "this",
  "predicate": {
    "minecraft:flags": {
      "is_sneaking": true
    }
  }
}
```
For versions before 26.3 the `type` field is renamed to `condition`.

## Commands

We can use the built-in scoreboard criteria of `minecraft:sneak_time`.

```mcfunction
# In chat
scoreboard objectives add sneak_time minecraft:sneak_time

# Command blocks
execute as @a[scores={sneak_time=1..}] run say Sneaking
scoreboard players reset @a sneak_time
```

### Crawling

The main approach to detect crawling is to check the size of the hitbox, since it decreases in vertical size. Keep in mind that flying with an elytra and swimming also produce the same hitbox so you will need to account for these scenarios too. You can watch a full tutorial by Infernal Device here: https://youtube.com/watch?v=uldA-_sUgpw.

[![Infernal Device's Video on crawling](https://img.youtube.com/vi/uldA-_sUgpw/0.jpg)](https://www.youtube.com/watch?v=uldA-_sUgpw)

## Sleeping
### Advancement

We can use the [`minecraft:slept_in_bed`](https://minecraft.wiki/w/Advancement_definition#minecraft:slept_in_bed) advancement criteria to detect when the player sleeps.

```json
# advancement example:slept_in_bed
{
  "criteria": {
    "slept_in_bed": {
      "trigger": "minecraft:slept_in_bed"
    }
  },
  "rewards": {
    "function": "example:slept_in_bed"
  }
}
```
```mcfunction
# function example:slept_in_bed
advancement revoke @s only example:slept_in_bed
say I started sleeping
```

### Commands

We can use the built-in `minecraft:time_since_rest` scoreboard criteria. If it is 0, then the player is currently sleeping. You can use the non-destructive method of detecting when the score changed if you only want to run a command once. To avoid this triggering the first time the player joins the game, we can use the `minecraft:play_time` scoreboard criteria.


```mcfunction
# In chat
scoreboard objectives add time_since_rest minecraft:time_since_rest
scoreboard objectives add play_time minecraft:play_time

# Command block
execute as @a[scores={time_since_rest=0,scores=play_time=1..}] run say sleeping
```

## Bedrock
For bedrock, check this detailed article https://wiki.bedrock.dev/commands/detect-movements.