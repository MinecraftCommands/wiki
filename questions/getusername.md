# Get the player username


## Text components

This method only works if you want to display a playername in chat, title, subtitle, actionbar, in a sign or other places where text components are valid (and can be [resolved](https://minecraft.wiki/w/Text_component_format#Component_resolution)). For example, showing the name of the nearest player in the chat. Keep in mind that if no entity matches the selector it will default to an empty string. If multiple entities match the selector it will be displayed as "Player1, Player2, Player3".

```mcfunction
# Java
tellraw @a {"selector":"@p"}
# Bedrock
tellraw @a {"rawtext":[{"selector":"@p"}]}
```

## Loot table

| 📝 Note |
|This only works in java edition|

The above method works well for messages but won't work for all cases (like for example, storing the player name in a storage). To solve this, we can summon an item display and use the `fill_player_head` loot table to insert the player head in the entity. Then we can run any function as a macro with the item display `profile` component. Lastly, we kill the item display since we don't need it anymore.

```mcfunction
# function example:get_username
tag @s add this
execute at @s summon item_display run function example:get_username/item_display
tag @s remove this

# function function example:get_username/item_display
tag @s add this
execute as @a[tag=this] loot replace entity @e[type=item_display,tag=this] container.0 loot {pools:[{rolls:1,entries:[{type:"minecraft:item",name:"player_head",functions:[{function:"minecraft:fill_player_head",entity:"this"}]}]}]}
function example:get_username/result with entity @e[type=item_display,tag=this] item.components."minecraft:profile"
kill @s

# function example:get_username/result
$say $(name)
```

For 1.20.2-1.20.4 it requires some small changes, since components don't exist and tags are used instead, but the approach is the same.

<details markdown="1">
  <summary style="color: #e67e22; font-weight: bold;">See datapack</summary>

```mcfunction
# function example:get_username
tag @s add this
execute summon item_display run function example:get_username/item_display
tag @s remove this

# function function example:get_username/item_display
tag @s add this
execute as @a[tag=this] loot replace entity @e[type=item_display,tag=this] container.0 loot example:player_head
function example:get_username/result with entity @e[type=item_display,tag=this] item.tag.SkullOwner
kill @s

# function example:get_username/result
$say $(Name)
```
```json
# loot_table example:player_head
{
  "pools": [
    {
      "rolls": 1,
      "entries": [
        {
          "type": "minecraft:item",
          "name": "player_head",
          "functions": [
            {
              "function": "minecraft:fill_player_head",
              "entity": "this"
            }
          ]
        }
      ]
    }
  ]
}
```
</details>