# Detect the player killing entities and players

_Related: [Detect Player Deaths](/wiki/questions/playerdeaths)_

  - [Java](#java)
  - [Bedrock](#bedrock)
    - [Without behavior packs](#without-behavior-packs)


## Java

In Java this is easy, as there is a whole lot of [scoreboard objective criteria](https://minecraft.wiki/Scoreboard#Criteria) for every entity you can kill in the game, using the format `minecraft.killed:minecraft.<entity>`, where `<entity>` is a valid type of entity.
This method for command blocks cannot check any NBT data, including the tag of the killed mob. 

But this can be done using advancement using the [`player_killed_entity`](https://minecraft.wiki/wiki/Advancement/JSON_format#minecraft:player_killed_entity) advancement trigger, if you're using a datapack that looks like this:

```json
{
  "criteria": {
    "requirement": {
      "trigger": "minecraft:player_killed_entity",
      "conditions": {
        "entity": {
          "type": "minecraft:pig",
          "nbt": "{Tags:[\"example\"]}"
        }
      }
    }
  },
  "rewards": {
    "function": "example:kill_entity"
  }
}
```

The advancement from above will check if the entity that the player killed has that NBT (so it has the tag `example` in this case); if you don't want to check NBT you can just use:

```json
{
  "criteria": {
    "requirement": {
      "trigger": "minecraft:player_killed_entity",
      "conditions": {
        "entity": {
          "type": "minecraft:pig"
        }
      }
    }
  },
  "rewards": {
    "function": "example:kill_entity"
  }
}
```

When the player kills a pig, they will run the function `example:kill_entity` as it is set as a reward.

## Bedrock

In Bedrock this is much more tricky, as there is only the dummy objective type.

The best way this Subreddit has come up with so far is to use a modified loot table, making the entity drop a certain item on death that they otherwise wouldn't drop. _This also works for players, but only if the `keepInventory` gamerule is false._ To change an entities loot table, you will have to modify their behavior file and change the loot table to be your custom one or replace their default loot table if they have one. Find the `minecraft:loot` component (or add it if it doesn't exist) and give it your custom loot table.

```json
"minecraft:loot": {
    "table": "loot_tables/<your/loot/table>.json"
},
```

Now, you of course need to create said loot table as well. Here is an example of a loot table that will always drop 1 dragon egg if the entity is killed by a player:

<details markdown="1">
  <summary style="color: #e67e22; font-weight: bold;">See example</summary>

```json
{
    "pools": [
        {
            "conditions": [
                {
                    "condition": "killed_by_player_or_pets"
                }
            ],
            "rolls": 1,
            "entries": [
                {
                    "type": "item",
                    "name": "minecraft:dragon_egg",
                    "weight": 1
                }
            ]
        }
    ]
}
```

</details>

_This will overwrite the normal loot table for the entity. If you want the item to be dropped in addition to the normal loot, you will have to modify the entity's standard loot table._

From here you can go two ways:  

1. Either you give the player closest to the dropped item the credit for the kill by counting their score up by one, then kill the item. Be aware that this can easily create false results if the combat is ranged or in a chaotic environment, where the closest player to the killed player isn't necessarily the killer.  
2. Or you require the players to pick up the item for their kill to be counted, check for the item in their inventory, count up by 1 if you find it and clear 1 from their inventory.  
Read [here](/wiki/questions/detectitem) how you can detect a certain item.

_Credit to [u/DanRileyCG](https://www.reddit.com/user/DanRileyCG/), for the original [thread here](https://www.reddit.com/r/MinecraftCommands/comments/f7jd9f/help_with_server/)_

### Without behavior packs

**If you don't have access to the files and are thus unable to create a behavior pack for this, there is a toned down version for player kills only that you can do.**

It involves giving the player the special item into their inventory and making sure they drop it on death. To make it fool-proof / non-abuseable, there are a few more systems required (don't repeatedly replace it, or they could farm kills, only replace it if it's empty, don't award a kill if the player does have a bee nest in their inventory, etc.). Too much to go into in this wiki article, but here is a video where they show off one such system: [https://www.youtube.com/watch?v=zuhd3qEOJ1I](https://www.youtube.com/watch?v=zuhd3qEOJ1I). Keep in mind that this video uses the old `execute` syntax and it will need to be updated, see [this information](https://wiki.bedrock.dev/commands/new-execute.html) about the changes