# Do custom crafting (with NBT)

* [Java](#bedrock-edition-addon)
* [Bedorck](#bedrock-edition-addon)

## Java
The vanilla crafting system does not support NBT data. Although 1.20.5 added the ability to create recipes with custom crafting data, it still does not support custom data for ingredients. This article provides several ways to create custom crafts that support custom items for both the result and the ingredients.

----

### Floor Crafting

The easiest way to create a custom craft is to use floor crafting. You can create crafting recipes with any ingredients and for each recipe you only need 1 command block, which makes it easy to create many recipes without creating an excessive load on the server.

Below is a detailed example of creating a floor crafting command.

First, let’s introduce custom ingredients and how much of each ingredient is needed:

    # Example custom ingredients
    give @s emerald{some:true,display:{Name:'{"text":"Some Emerald","italic":false}'}}
    give @s diamond{custom:true,display:{Name:'{"text":"Some Diamond","italic":false}'}}
    give @s redstone{data:true,display:{Name:'{"text":"Some Redstone","italic":false}'}}

    Example recipe:
    2x emerald
    1x diamond
    5x redstone

It is highly recommended for all custom ingredients in a recipe to use [items with custom tags](/wiki/questions/customitemtag) because the command will be very long anyway, so this is a good way to shorten the command length and make the command easier to read/edit. Also check out the article on how to [detect custom tags](/wiki/questions/detectitem).

Below is the command in pseudocode:

    execute at @a as <first_ingredient> at @s store success entity @s Age short 6000 store success entity <second_ingredient> Age short 6000 store success entity <third_ingredient> Age short 6000 ... run summon <craft_result>

For optimization, so as not to check all the items in the world, `<first_ingredient>` should be selected in a small radius around the player, approximately 6 blocks. Next, use the `<first_ingredient>` position for the rest of the command. After you need store the success of the command in the data of this item, the `Age` tag multiplied by 6000 will instantly despawn this item if crafting is successful. All subsequent ingredients must be immediately selected inside the target selector in the `store success entity` subcommand. Be sure to specify for each further ingredient in the target selector `limit=1` and a small `distance`, approximately 0.5 blocks. It might also make sense to add an `OnGround:true` check for each ingredient so that the crafting only works when the items land nearby.

From all this, can now assemble a ready-made command. Here is an example of a simple floor crafting command that will work on versions 1.13 - 1.20.4:

    # Command block / tick function
    execute at @a as @e[type=item,distance=..6,nbt={OnGround:true,Item:{id:"minecraft:emerald",Count:2b,tag:{some:true}}}] at @s store success entity @s Age short 6000 store success entity @e[type=item,distance=..0.5,limit=1,nbt={OnGround:true,Item:{id:"minecraft:diamond",Count:1b,tag:{custom:true}}}] Age short 6000 store success entity @e[type=item,distance=..0.5,limit=1,nbt={OnGround:true,Item:{id:"minecraft:redstone",Count:5b,tag:{data:true}}}] Age short 6000 run summon item ~ ~ ~ {Item:{id:"minecraft:ender_eye",Count:1b,tag:{custom_result:true,display:{Name:'{"text":"Some Custom Result","italic":false}'}}}}

**Note:** As you can understand, you can use not only items, but also any entities as a crafting ingredient, just as the result of crafting can be any command executed at this coordinate.

But you can also spice up the crafting a bit with some animations, but this raises the required Minecraft version to 1.19.4 - 1.20.4.

First, let's create an animation for the crafting result to bounce in a random direction. This is based on the fact that if shulker\_box takes damage and is destroyed, it will drop all its contents (1.17+). The easiest way to damage shulker\_box summon is as an item with the `Fire:20s` tag:

    summon item ~ ~ ~ {Fire:20s,Item:{id:"minecraft:shulker_box",Count:1b,tag:{BlockEntityTag:{Items:[{Slot:0b,id:"minecraft:stone",Count:1b}]}}}}

This is also an easy way to give players not just one item, but multiple items as a result of crafting.

However, this creates a small problem - if the craft is inside water blocks, then the shulker\_box will be extinguished and will not drop the contents and the player will receive a free shulker\_box. To solve this problem, you can add to any part of the command a check that the current block is an air block.

You can also add particles/sound when crafting; to do this, at the end of the command (before `run`) you need to place this code and create a scoreboard objective `craft_anim` (1.19.4+):

    execute ... summon area_effect_cloud store success score @s craft_anim run ...

And add several command blocks that will show particles and play sound:

    execute at @e[type=area_effect_cloud,scores={craft_anim=1}] run particle minecraft:reverse_portal ~ ~.2 ~ 0 0 0 0.025 100
    execute at @e[type=area_effect_cloud,scores={craft_anim=1}] run playsound minecraft:entity.evoker.prepare_wololo neutral @a

**Note: These commands must be strictly at the end of the chain of command blocks with floor crafting recipes!** If you have several recipes, then these commands should be after all the recipes.

And now here is a complete example for floor crafting which uses only one command block for each recipe and two command blocks for crafting animation, common to all crafts, working on versions 1.19.4 - 1.20.4:

    # Command blocks / tick function
    execute at @a as @e[type=item,distance=..6,nbt={OnGround:true,Item:{id:"minecraft:emerald",Count:2b,tag:{some:true}}}] at @s if block ~ ~ ~ air store success entity @s Age short 6000 store success entity @e[type=item,distance=..0.5,limit=1,nbt={OnGround:true,Item:{id:"minecraft:diamond",Count:1b,tag:{custom:true}}}] Age short 6000 store success entity @e[type=item,distance=..0.5,limit=1,nbt={OnGround:true,Item:{id:"minecraft:redstone",Count:5b,tag:{data:true}}}] Age short 6000 summon area_effect_cloud store success score @s craft_anim run summon item ~ ~ ~ {Fire:20s,Item:{id:"minecraft:shulker_box",Count:1b,tag:{BlockEntityTag:{Items:[{Slot:0b,id:"minecraft:ender_eye",Count:1b,tag:{display:{Name:'{"text":"Some Custom Result","italic":false}'}}}]}}}}
    execute at @e[type=area_effect_cloud,scores={craft_anim=1}] run particle minecraft:reverse_portal ~ ~.2 ~ 0 0 0 0.025 100
    execute at @e[type=area_effect_cloud,scores={craft_anim=1}] run playsound minecraft:entity.evoker.prepare_wololo neutral @a

**Note:** You can have as many crafting ingredients in one recipe as you want, you are only limited by the long command block length of 32k characters.

Since version 1.20.5 NBT tags have been replaced with [components](https://minecraft.wiki/w/Data_component_format), so the command for this is now slightly different:

    # Example custom ingredients
    give @s emerald[minecraft:custom_data={some:true},item_name='"Some Emerald"']
    give @s diamond[minecraft:custom_data={custom:true},item_name='"Some Diamond"']
    give @s redstone[minecraft:custom_data={data:true},item_name='"Some Redstone"']
    
    # Command blocks / tick function
    execute at @a as @e[type=item,distance=..6,nbt={OnGround:true,Item:{id:"minecraft:emerald",count:2,components:{"minecraft:custom_data":{some:true}}}}] at @s if block ~ ~ ~ air store success entity @s Age short 6000 store success entity @e[type=item,distance=..0.5,limit=1,nbt={OnGround:true,Item:{id:"minecraft:diamond",count:1,components:{"minecraft:custom_data":{custom:true}}}}] Age short 6000 store success entity @e[type=item,distance=..0.5,limit=1,nbt={OnGround:true,Item:{id:"minecraft:redstone",count:5,components:{"minecraft:custom_data":{data:true}}}}] Age short 6000 summon area_effect_cloud store success score @s craft_anim run summon item ~ ~ ~ {Fire:20s,Item:{id:"minecraft:shulker_box",count:1,components:{"minecraft:container":[{slot:0,item:{id:"minecraft:ender_eye",count:1,components:{"minecraft:item_name":'{"text":"Some Custom Result"}'}}}]}}}
    execute at @e[type=area_effect_cloud,scores={craft_anim=1}] run particle minecraft:reverse_portal ~ ~.2 ~ 0 0 0 0.025 100
    execute at @e[type=area_effect_cloud,scores={craft_anim=1}] run playsound minecraft:entity.evoker.prepare_wololo neutral @a

----

### Custom Dropper Crafting

Another popular way to create custom crafts that support NBT data is to use custom dropper crafting. This method uses a dropper / dispenser interface to simulate the crafting grid as the crafting\_table. This method is more difficult to implement, but looks much better than floor crafting.

In this article we will use an item_display to make a custom dropper look like a crafting_table, so this method will apply to versions 1.19.4 and above, although you can use a falling_block or something else for earlier versions.

    # 1.19.4 - 1.20.4
    give @p bat_spawn_egg{EntityTag:{id:"minecraft:item_display",Tags:["custom_crafting","placing"],Rotation:[0f,0f],brightness:{sky:10,block:10},transformation:{left_rotation:[0f,0f,0f,1f],right_rotation:[0f,0f,0f,1f],translation:[0f,0.5f,0f],scale:[1.01f,1.01f,1.01f]},item:{id:"minecraft:crafting_table",Count:1b}},display:{Name:'"Custom Crafting Table"'}}
    
    # 1.20.5+
    give @p bat_spawn_egg[entity_data={id:"minecraft:item_display",Tags:["custom_crafting","placing"],Rotation:[0f,0f],brightness:{sky:10,block:10},transformation:{left_rotation:[0f,0f,0f,1f],right_rotation:[0f,0f,0f,1f],translation:[0f,0.5f,0f],scale:[1.01f,1.01f,1.01f]},item:{id:"minecraft:crafting_table",count:1}},item_name='"Custom Crafting Table"']

This crafting method consists of two parts: the controller and the recipes.

The controller is a command block that detects the placement of a spawn_egg to install a dropper block, as well as to remove the support entity when the block is destroyed by the player and return a spawn_egg to the player.

    # Command blocks (controller) for 1.19.4 - 1.20.4
    execute at @e[type=item_display,tag=custom_crafting,tag=placing] run setblock ~ ~ ~ dropper[facing=up]{CustomName:'"Custom Crafting Table"'}
    tag @e[type=item_display,tag=custom_crafting,tag=placing] remove placing
    execute as @e[type=item_display,tag=custom_crafting] at @s unless block ~ ~ ~ dropper[facing=up] run data modify entity @e[type=item,distance=..1,nbt={Item:{id:"minecraft:dropper"}},limit=1] Item set value {id:"minecraft:bat_spawn_egg",Count:1b,tag:{EntityTag:{id:"minecraft:item_display",Tags:["custom_crafting","placing"],Rotation:[0f,0f],brightness:{sky:10,block:10},transformation:{left_rotation:[0f,0f,0f,1f],right_rotation:[0f,0f,0f,1f],translation:[0f,0.5f,0f],scale:[1.01f,1.01f,1.01f]},item:{id:"minecraft:crafting_table",Count:1b}},display:{Name:'"Custom Crafting Table"'}}}
    execute as @e[type=item_display,tag=custom_crafting] at @s unless block ~ ~ ~ dropper[facing=up] run kill @s
    
    # Command blocks (controller) for 1.20.5+
    execute at @e[type=item_display,tag=custom_crafting,tag=placing] run setblock ~ ~ ~ dropper[facing=up]{CustomName:'"Custom Crafting Table"'}
    tag @e[type=item_display,tag=custom_crafting,tag=placing] remove placing
    execute as @e[type=item_display,tag=custom_crafting] at @s unless block ~ ~ ~ dropper[facing=up] run data modify entity @e[type=item,distance=..1,nbt={Item:{id:"minecraft:dropper"}},limit=1] Item set value {id:"minecraft:bat_spawn_egg",count:1,components:{"item_name":'"Custom Crafting Table"',"minecraft:entity_data":{id:"minecraft:item_display",Tags:["custom_crafting","placing"],Rotation:[0f,0f],brightness:{sky:10,block:10},transformation:{left_rotation:[0f,0f,0f,1f],right_rotation:[0f,0f,0f,1f],translation:[0f,0.5f,0f],scale:[1.01f,1.01f,1.01f]},item:{id:"minecraft:crafting_table",count:1}}}}
    execute as @e[type=item_display,tag=custom_crafting] at @s unless block ~ ~ ~ dropper[facing=up] run kill @s

Recipes can be made as separate command blocks. Each recipe is a separate command block that checks the block data in the Items tag, and if the data matches, then replace the Items tag with your item as the crafting result.

**Hint:** If you find it difficult to create a command to check block data, then put your recipe in a dropper / dispenser. Exit the block interface and press `F3 + I` to copy the block data. You will receive a /setblock command with the full block data. This command has all of the blocks data, including data that is irrelevant or even hindering our ability to properly detect the relevant items. Thus you should not use the NBT data (`{}`) as-is, but remove any and all unnecessary NBT data. This involves removing all but the defining attributes of the item: `id`, `count`, `Slot` and potentially the custom data you used to mark your custom items.

Below is a schematic representation of the command to check the recipe:

    execute at @e[type=item_display,tag=custom_crafting] if block ~ ~ ~ dropper{Items:[<check_recipe>]} run data modify block ~ ~ ~ Items set value [{Slot:<slot>},<result_craft_data>]

Now you can create a ready-made command for the recipe. Below is an example of crafting without checking custom NBT tags for compactness, but you can add any NBT data for verification:

    # Example recipe
    [E][ ][E]
    [R][D][R]
    [R][R][R]
    
    E - minecraft:emerald
    D - minecraft:diamond
    R - minecraft:redstone
    
    # Command block (recipe) for 1.19.4 - 1.20.4
    execute at @e[type=item_display,tag=custom_crafting] if block ~ ~ ~ dropper{Items:[{Slot:0b,id:"minecraft:emerald",Count:1b}, {Slot:2b,id:"minecraft:emerald",Count:1b}, {Slot:3b,id:"minecraft:redstone",Count:1b}, {Slot:4b,id:"minecraft:diamond",Count:1b}, {Slot:5b,id:"minecraft:redstone",Count:1b}, {Slot:6b,id:"minecraft:redstone",Count:1b}, {Slot:7b,id:"minecraft:redstone",Count:1b}, {Slot:8b,id:"minecraft:redstone",Count:1b}]} run data modify block ~ ~ ~ Items set value [{Slot:4b,id:"minecraft:ender_eye",Count:1b,tag:{custom_result:true,display:{Name:'{"text":"Some Custom Result","italic":false}'}}}]
    
    # Command block (recipe) for 1.20.5+
    execute at @e[type=item_display,tag=custom_crafting] if items block ~ ~ ~ container.0 emerald[custom_data~{some:true}] unless items block ~ ~ ~ container.1 * if items block ~ ~ ~ container.2 emerald[custom_data~{some:true}] if items block ~ ~ ~ container.3 redstone[custom_data~{data:true}] if items block ~ ~ ~ container.4 diamond[custom_data~{custom:true}] if items block ~ ~ ~ container.5 redstone[custom_data~{data:true}] if items block ~ ~ ~ container.6 redstone[custom_data~{data:true}] if items block ~ ~ ~ container.7 redstone[custom_data~{data:true}] if items block ~ ~ ~ container.8 redstone[custom_data~{data:true}] run data modify block ~ ~ ~ Items set value [{Slot:4b,id:"minecraft:ender_eye",count:1b,components:{"minecraft:custom_data":{custom_result:true},"minecraft:item_name":'"Some Custom Result"'}}]

**Note:** Empty slots that do not contain a recipe item will be ignored, so if the player leaves any items in these slots, then these items will be deleted. To avoid this, you can add, in addition to checking the block data, to check that there are no items in the empty slots using the `unless data block` subcommand. Each slot requires a separate subcommand:

    execute at @e[type=item_display,tag=custom_crafting] if block ~ ~ ~ dropper{Items:[<check_recipe>]} unless data block ~ ~ ~ Items[{Slot:<empty_slot>}] unless data block ~ ~ ~ Items[{Slot:<empty_slot>}] run data modify ...

----

### Custom Knowledge Book Crafting

#### Before 1.20

Before version 1.20 you could create custom crafts using the Knowledge Book method, however **this method does not support creating custom crafts with NBT data for ingredients, only for the result**.

Typically, many files are created for each craft: a recipe file, an advancement with the recipe\_unlocked trigger, and a function file, inside of which the knowledge\_book is deleted, the recipe and the advancement are revoked, and the item with custom data is given. However, if you add a lot of recipes, it quickly becomes unwieldy.

However, it can be done more compactly and cleaner. You only need to create one function to take all custom recipes, revoke all recipe advancements and clear the knowledge_book. You can also have one root advancement for all recipes to make revoking them easier. So for each recipe you only need the recipe file, the individual advancement and a loot table with the custom item.

It works like this:

When you craft an item, the corresponding advancement recipe immediately gives you the desired item through the loot table and launches the recipe and advancements reset function. In this function you need to take every recipe in your datapack and revoke all recipe advancements **from** root recipe advancement, for example:

    advancement revoke @s from example:recipe/root

And when creating all recipe advancements, you use recipe root advancement as the parent. This allows you to revoke all recipe advancements with one command.

Below is an example of how to implement this approach:

    # recipe example:some_custom_result (1.15-1.20.4)
    {
      "type": "minecraft:crafting_shaped",
      "pattern": [
        "E E",
        "RDR",
        "RRR"
      ],
      "key": {
        "E": {
          "item": "minecraft:emerald"
        },
        "D": {
          "item": "minecraft:diamond"
        },
        "R": {
          "item": "minecraft:redstone"
        }
      },
      "result": {
        "item": "minecraft:knowledge_book"
      }
    }
    
    # loot_table example:some_custom_result (1.15-1.20.4)
    {
      "pools": [
        {
          "rolls": 1,
          "entries": [
            {
              "type": "minecraft:item",
              "name": "minecraft:ender_eye",
              "functions": [
                {
                  "function": "minecraft:set_name",
                  "entity": "this",
                  "name": {
                    "text": "Some Custom Result",
                    "italic": false
                  }
                },
                {
                  "function": "minecraft:set_nbt",
                  "tag": "{custom_result:true}"
                }
              ]
            }
          ]
        }
      ]
    }
    
    # advancement example:recipe/root
    {
      "criteria": {
        "root_recipe": {
          "trigger": "minecraft:impossible"
        }
      }
    }
    
    # advancement example:recipe/some_custom_result
    {
      "parent": "example:recipe/root",
      "criteria": {
        "requirement": {
          "trigger": "minecraft:recipe_unlocked",
          "conditions": {
            "recipe": "example:some_custom_result"
          }
        }
      },
      "rewards": {
        "function": "example:recipe_reset",
        "loot": [
          "example:some_custom_result"
        ]
      }
    }
    
    # function example:recipe_reset
    advancement revoke @s from example:recipe/root
    clear @s minecraft:knowledge_book
    recipe take @s example:some_custom_result
    recipe take @s example:another_custom_result
    recipe take @s example:more_custom_result
    ...

The obvious disadvantages of this method are the inability to create a craft with custom ingredients, and if you give yourself all the recipes, it will give you all the custom items, but you still won’t be able to have your custom crafts in the recipe book.

#### After 1.20

Starting with version 1.20, a new advancement trigger was added - `recipe_crafted`. This one triggers when you have crafted the specified craft, but not just unlocked it. Therefore you don't need to take recipes and you can have your crafts in the craft book, although these crafts will appear as knowledge\_book.

You can now use this advancement trigger to check the NBT data of the ingredients. For this you can use the `ingredients` condition. Each entry will correspond to exactly one item. So, if you use several identical custom items in a recipe, then you need to specify this item several times. The ingredient check only checks the specified ingredients, other items in the recipe will be ignored.

Below is an example for creating an advancement for a custom craft, which must have ingredients with NBT data to get the craft result. Only the changes in this version are shown here, the rest of the code is unchanged:

    # advancement example:recipe/some_custom_result
    {
      "parent": "example:recipe/root",
      "criteria": {
        "requirement": {
          "trigger": "minecraft:recipe_crafted",
          "conditions": {
            "recipe_id": "example:some_custom_result",
            "ingredients": [
              {
                "items": [
                  "minecraft:emerald"
                ],
                "nbt": "{some:true}"
              },
              {
                "items": [
                  "minecraft:emerald"
                ],
                "nbt": "{some:true}"
              },
              {
                "items": [
                  "minecraft:diamond"
                ],
                "nbt": "{custom:true}"
              },
              {
                "items": [
                  "minecraft:redstone"
                ],
                "nbt": "{data:true}"
              },
              {
                "items": [
                  "minecraft:redstone"
                ],
                "nbt": "{data:true}"
              },
              {
                "items": [
                  "minecraft:redstone"
                ],
                "nbt": "{data:true}"
              },
              {
                "items": [
                  "minecraft:redstone"
                ],
                "nbt": "{data:true}"
              },
              {
                "items": [
                  "minecraft:redstone"
                ],
                "nbt": "{data:true}"
              }
            ]
          }
        }
      },
      "rewards": {
        "function": "example:recipe_reset",
        "loot": [
          "example:some_custom_result"
        ]
      }
    }
    
    # function example:recipe_reset
    advancement revoke @s from example:recipe/root
    clear @s minecraft:knowledge_book

**Important!** The player can still use regular items in crafting, but then the advancement will not work and the player will only craft the knowledge\_book. To avoid this, you need to create a more complex crafting system.

#### 1.20.5 and above

Version 1.20.5 also added the ability to create crafts with custom data, but only for the craft result - NOT for ingredients. Therefore, if you want to use custom items in crafting, then in this version it will be the same as described for the previous version.

If you only need a custom item as a result of crafting, then now you can use only one recipe file and you do not need to use advancements and functions:

    # recipe example:some_custom_result
    {
      "type": "minecraft:crafting_shaped",
      "pattern": [
        "E E",
        "RDR",
        "RRR"
      ],
      "key": {
        "E": {
          "item": "minecraft:emerald"
        },
        "D": {
          "item": "minecraft:diamond"
        },
        "R": {
          "item": "minecraft:redstone"
        }
      },
      "result": {
        "id": "minecraft:ender_eye",
        "components": {
          "minecraft:item_name": "'Some Custom Result'",
          "minecraft:custom_data": {"custom_result": true }
        }
      }
    }

**Note:** If you use a custom tag for an item like `custom:1`, then the recipe will give you an item with the tag `custom:1b` due to the conversion of JSON to NBT format.

## Bedrock edition addon
In bedrock edition, you can use addons to make custom recepies, all recipes are stored in the recipes folder in the behavior pack root.

You can make a custom recepie for the following blocks:
* Crafting:
  * crafting_table
  * stonecutter
  * smithing_table
* Cooking and Smelting:
  * furnace
  * blast_furnace
  * smoker
  * campfire
  * soul_campfire
* Brewing:
  * brewing_stand
* Education:
  * material_reducer

You can also make [custom crafting tables](https://wiki.bedrock.dev/blocks/block-components.html#crafting-table)

Here is an example of a shaped recipe from the bedrock wiki:

	{
		"format_version": "1.17.41",
		"minecraft:recipe_shaped": {
			"description": {
				"identifier": "wiki:cold_steel_sword"
			},
			"tags": ["crafting_table", "altar"],
			"pattern": [
				"X",
				"X",
				"I"
			],
			"key": {
				"X": "wiki:cold_steel",
				"I": "minecraft:stick"
			},
			"unlock": [
				{
					"item": "wiki:cold_steel"
				},
				{
					"item": "minecraft:wool",
					"data":  3
				},
				{
					"context": "PlayerInWater"
				}
			],
			"result": "wiki:cold_steel_sword"
		}
	}


A quick explanation:
* `format_version` is the minecraft version, it is recomended to use the last relase.
* The `description` holds the holds the `identifier` of a recipe
* `tags` are the list of interfaces were the crafting can be used
* `pattern` is the position of the ingridients in order to get the result
* `key` specifies what item is fro every letter from `pattern`
* `unlock` specifies what will unlock the recipe
* `result` specifies what item you will get when crafting this recipe

You can find a detailed tutorial [in the bedrock wiki](https://wiki.bedrock.dev/loot/recipes.html) that contains information about other blocks such as furnances or brewing stands.
