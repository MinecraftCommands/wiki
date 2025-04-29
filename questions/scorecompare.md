# Check if a score / storage is equal to, greater than, or less than, another score / storage

## Scoreboard

### 1.13 and above

In 1.13, the [`if score`](https://minecraft.wiki/w/Commands/execute#(if%7Cunless)_score) execute subcommand makes this a lot easier. The syntax you'll want is:

    execute if score <target> <targetObjective> (<|<=|=|>|>=) <source> <sourceObjective> run <command>

For example, to check if the executer's `kills` is greater than their `deaths`:

    execute if score @s kills > @s deaths run say I have more kills than deaths!

Or, to check if the nearest player's `money` score is equal to their `cost`:

    execute if score @p money = @p cost run <command>

You can also use `matches` to check for a [range](/wiki/questions/range) if you want to always check for a given range and don't want to store that in a different scoreboard:

    execute if score @s diamonds matches 1..4 run say I have somewhere between 1 and 4 diamonds in my inventory.
    execute if entity @s[scores={diamonds=1..4}] run say I have somewhere between 1 and 4 diamonds in my inventory.


### 1.12 and below

To do this, we must take one score from another, check if the score is now equal to/greater than/less than 0, then add the score back (to restore the first score's original value).

For example, select all players whose `kills` score is greater than their `deaths` score:

1. Take everyone's `deaths` from their `kills`

       execute @a ~ ~ ~ scoreboard players operation @s kills -= @s deaths

2. Select all players that now have a positive `kills` score (so they had more kills than deaths)

       say @a[score_kills_min=1]

3. Add everyone's `deaths` back to their `kills`, to restore `kills` original value

       execute @a ~ ~ ~ scoreboard players operation @s kills += @s deaths

       execute if score @p money matches 10.. run say I have 10 money or more.

## Predicate / storage

### 1.20.5 and above

Since version 1.20.5 can also compare values in storage directly, without copying values to the scoreboard. Below is an example of using the `minecraft:value_check` condition to compare values in storage.

<details>
  <summary style="color: #e67e22; font-weight: bold;">See example</summary>

```mcfunction
# Example storage
data merge storage example:data {value:7.5f,min:0,max:10}

```json
# predicate example:storage_compire
{
  "condition": "minecraft:value_check",
  "value": {
    "type": "minecraft:storage",
    "storage": "example:data",
    "path": "value"
  },
  "range": {
    "min": {
      "type": "minecraft:storage",
      "storage": "example:data",
      "path": "min"
    },
    "max": {
      "type": "minecraft:storage",
      "storage": "example:data",
      "path": "max"
    }
  }
}
```

</details>

This now allows to compare values more accurately because it supports non-integer variable values for comparison.

### 1.15 and above

Since version 1.15 you can also use [predicates](https://minecraft.wiki/w/Predicate) in a datapack to compare scores. Unlike using the `if score` subcommand, you cannot, in most cases, compare the score values between two entities. Basically this is only available in mob loot tables and you can only compare the score between the killed mob (`this`) and the entity that killed the mob (`killer` / `killer_player`), or the projectile that killed the mob (`direct_killer`). In other cases, you can only check the score of the selected entity (`this`) and score [fakename](/wiki/questions/fakeplayer).

You can compare scores in a predicate using the `minecraft:entity_scores` condition to compare the score of the selected entity with a specific value or a specified range of values, as well as using the `minecraft:value_check` condition, which does the same thing, but without using the entity.

But you can't just use comparison operators (=, >, <, >=, <=), but only compare whether the value is in the specified range.

For `entity_score` condition, you can compare the value of the selected entity with an exact value specified manually or a range (`min` and `max`). When using range can use for each min and max:

- exact value (supports non-integer values)
- [binomial distribution](https://en.wikipedia.org/wiki/Binomial_distribution)
- score
- storage (1.20.5+)

Example to compare that players score `kills` >= score `deaths`:

<details>
  <summary style="color: #e67e22; font-weight: bold;">See example</summary>

```json
# execute if score @s kills >= @s deaths
{
  "condition": "minecraft:entity_scores",
  "entity": "this",
   "scores": {
     "kills": {
      "min": {
        "type": "minecraft:score",
        "target": "this",
        "score": "deaths"
      }
    }
  }
}
```
</details>

For the <= operator, simply replace `"min"` with `"max"` in the predicate above.

But if you want to check that score `kills` > `deaths`, then checking in the predicate will be a little more complicated. So, we need to do two checks: first check that `kills` >= `deaths`, AND the second check is the inversion of the condition `kills` <= `deaths`.

<details>
  <summary style="color: #e67e22; font-weight: bold;">See example</summary>

```json
# execute if score @s kills > @s deaths
[
  {
    "condition": "minecraft:entity_scores",
    "entity": "this",
    "scores": {
      "kills": {
        "min": {
          "type": "minecraft:score",
          "target": "this",
          "score": "deaths"
        }
      }
    }
  },
  {
    "condition": "minecraft:inverted",
    "term": {
      "condition": "minecraft:entity_scores",
      "entity": "this",
      "scores": {
        "kills": {
          "max": {
            "type": "minecraft:score",
            "target": "this",
            "score": "deaths"
          }
        }
      }
    }
  }
]
```

</details>

If you need to check that `kills` = `deaths` score, then you can do one range check, where `"min"` and `"max"` are the same score.

<details>
  <summary style="color: #e67e22; font-weight: bold;">See example</summary>

```json
# execute if score @s kills = @s deaths
{
  "condition": "minecraft:entity_scores",
  "entity": "this",
  "scores": {
    "kills": {
      "min": {
        "type": "minecraft:score",
        "target": "this",
        "score": "deaths"
      },
      "max": {
        "type": "minecraft:score",
        "target": "this",
        "score": "deaths"
      }
    }
  }
}
```