# Item Click Detection

* [Java](#java)
* [Java and Bedorck](#java-and-bedrock)

## Java

For item clicks we have to differentiate between leftclick and rightclick detection in the hotbar / offhand as well as clicking on the item while it is in the inventory. Clicking on the item while it is in the inventory works since version 1.20.5 with the addition of a player cursor slot (`player.cursor`) for checking.

### Left / right clicks in / on specific areas

#### Interaction entity

Added in 1.19.4, the [interaction entity](https://minecraft.wiki/w/Interaction) now makes it easier to detect left/right clicks as it uses fewer commands and the hitbox size can be adjusted, making it less likely to miss a click, but otherwise has the same disadvantages as the [Hurt entity method](#hurt-entity).

Here's a simple example for command blocks:

```mcfunction
# Setup
summon minecraft:interaction ~ ~ ~ {Tags:["click_scan"],width:0.5f,height:0.5f}

# Command blocks
execute as @e[type=interaction,tag=click_scan] store success entity @s attack.player[] int 0 on attacker run say Left Click!
execute as @e[type=interaction,tag=click_scan] store success entity @s interaction.player[] int 0 on target run say Right Click!
```

**Note:** Do not create an interaction entity that is too large, otherwise click detection will be inconsistent.

| 💡 Tip |
|--------|
|To see interaction entity press `F3 + B` to show hitboxes|

If you need to check left/right clicks in a large area (or anywhere), then use multiple interaction entities or create a separate interaction entity for each player and teleport to the player every tick, and use the [scoreboard ID system](/wiki/questions/linkentity) for linking.

You can also check what the player is holding in his hand, but then removing the `interaction` / `attack` tag must be done in a separate command:

```mcfunction
# Command blocks
### 1.19.4 - 1.20.4
execute as @e[type=interaction,tag=click_scan] on target if entity @s[nbt={SelectedItem:{tag:{right_click:true}}}] run say Right Click!

### 1.20.5+
execute as @e[type=interaction,tag=click_scan] on target if items entity @s weapon *[custom_data~{right_click:true}] run say Right Click!
execute as @e[type=interaction,tag=click_scan] run data remove entity @s interaction
```

When using a datapack, you don't have to run these commands in a tick function, but only once when interacting using advancements:
* Right click - `minecraft:player_interacted_with_entity` advancement trigger
* Left click - `minecraft:entity_hurt_player` advancement trigger

<details>
  <summary style="color: #e67e22; font-weight: bold;">See example</summary>

```json
# advancement example:interaction/right_click
{
  "criteria": {
    "requirement": {
      "trigger": "minecraft:player_interacted_with_entity",
      "conditions": {
        "entity": [
          {
            "condition": "minecraft:entity_properties",
            "entity": "this",
            "predicate": {
              "type": "minecraft:interaction",
              "nbt": "{Tags:['click_scan']}"
            }
          }
        ]
      }
    }
  },
  "rewards": {
    "function": "example:click/right"
  }
}
```
```mcfunction
# function example:click/right
advancement revoke @s only example:interaction/right_click
say Right Click!
execute as @e[type=interaction,tag=click_scan] run data remove entity @s interaction
```
</details>

### Left-click

#### Hurt entity

This one is as easily explained as it is flawed: You can only detect left clicks, if you put something in front of the player to hit. Teleport some form of entity or mob that can take damage and has a hitbox directly in the players face or even over their head, so they have no other chance but to hit that entity. You can then "detect" the clicks either using an advancement with the `player_hurt_entity` trigger ([see here](https://minecraft.wiki/Advancements/JSON_format#minecraft:player_hurt_entity)) or with a scoreboard objective of type `minecraft.custom:minecraft.damage_dealt` ([see here](https://minecraft.wiki/Scoreboard#Criteria)). This method however has many obvious flaws:  

- you're blocking the player from hitting anything else but the entity in their face  
- you're also blocking the player from building and breaking blocks  
- due to the 20Hz nature of commandblocks/functions, if the game is slightly laggy or the player is moving fast, the entity will "lag" behind, leading to unreliable detection.  

While you can use this method more reliably if this is about hitting some special mobs with a special item or something (the advancement method is really good at that), **general leftclick detection is discouraged unless you have a very controlled environment**.

### Right-click

For rightclick detection we have [a lot of different ways](https://i.imgur.com/8gKEdp1.png) (image by [u/Dieuwt](https://www.reddit.com/u/Dieuwt), does not include 1.20.5+ methods), and different situations might call for different solutions. Since we cannot write a detailed guide on all these methods, we'll only describe the most common solutions here. A rundown of the knowledge book method can be found [here](https://www.reddit.com/r/MinecraftCommands/comments/g4jxzy/simple_rightclick_detection_without_sacrificing) (by [u/U2106_Later](https://www.reddit.com/u/U2106_Later)).

#### Carrot on a stick method

_Note: This works the same with the warped fungus on a stick._

Mapmakers have for many years now exploited what is technically [a bug](https://bugs.mojang.com/browse/MC-112991): Carrots on sticks have the property that when you click with one in your main hand or nothing clickable in your main hand with the CoaS in your offhand, it increases a player's "used carrot_on_a_stick" score (`minecraft.used:minecraft.carrot_on_a_stick` see [here](https://minecraft.wiki/Scoreboard#Criteria)). Testing if a player is holding a carrot on a stick with a specific tag like `{shoot_fireball:1b}` and also has `[scores={used_CoaS=1..}]` will test if a player clicks with a custom carrot on a stick. (related: [detect what item a player is holding](/wiki/questions/detectitem))

This is the generally preferred method of doing this if it fits your situation, as its pros outweigh the cons.

It should be noted that the debug_stick has the same property in which clicking with it can be tested using scoreboards but it also changes block properties and that tends to remove it from the list of reasonable methods. Since 1.16 there is also the warped fungus on a stick, which has the same properties as the carrot on a stick.

Pros:
- Multiplayer friendly.
- Unexploitable.

Cons:
- Will attract pigs/boost pigs speed when riding a pig. _(or striders if using the wfoas instead)_
- Looks like a carrot on a stick (fix see below).
- Only in pre-1.20.5: Has unremovable Damage NBT tag. This can be somewhat negated by making the CoaS unbreakable.

##### Make the CoaS look like any item

###### 1.13+

People tend to use a carrot on a stick and then use a resource pack to remodel them for various CustomModelData tags. This way the CoaS looks like your desired item while still providing the same rightclick functionality.

The `models/item/carrot_on_a_stick.json` file within the resource pack might end up looking like this:

<details>
  <summary style="color: #e67e22; font-weight: bold;">See file</summary>

```json
{
    "parent": "item/handheld",
        "textures": {
        "layer0": "item/carrot_on_a_stick"
    },
   
    "overrides": [
        {"predicate": {"custom_model_data":1}, "model": "item/blaze_powder"},
        {"predicate": {"custom_model_data":2}, "model": "item/nether_star"},
        {"predicate": {"custom_model_data":3}, "model": "item/blaze_rod"},
        {"predicate": {"custom_model_data":4}, "model": "item/black_dye"}
    ]
}
```
</details>

| 💡 Tip |
|---------|
|You can make the item completly invisible if using the custom model data of a chest|

###### 1.21.2+

The new `item_model` component allows you to make any item look like any other item without a resource pack. See this example command of a carrot on a stick that looks like a nether star

    give @p carrot[item_model="nether_star"] 1

#### Make item food method

##### 1.20.5+

From version 1.20.5 you can add a right click check for any item that does not already use a right click event.

This method involves adding the [`minecraft:food component`](https://minecraft.wiki/w/Data_component_format#food) to the item.

When using only command blocks, you need to actually consume this item in order for the usage statistics of your item to change. So it might make sense to set the `eat_seconds` tag to a small value, such as 0.05 seconds (1 tick). Here is a small example for command blocks:

```mcfunction
# Setup
give @s minecraft:stick[minecraft:food={nutrition:0,saturation:0f,eat_seconds:0.05f,can_always_eat:true}]
scoreboard objectives add click.stick used:stick

# Command blocks
execute as @a[scores={click.stick=1..}] run say Right Click!
scoreboard players reset @a click.stick
```

This method has obvious disadvantages, such as particles appearing when used, sounds and the fact that the item is actually used.

But when using a datapack, there are none of these disadvantages. Then you want to change `eat_seconds` to something very large so that the eating animation can't start and use the advancement trigger [`minecraft:using_item`](https://minecraft.wiki/w/Custom_advancement#minecraft:using_item) to check the item's usage. Since this advancement trigger is triggered every tick while the player is using the item, you can execute the command up to 20 times per second. However, often you don't want to do this as often and want to add a delay between command runs.
Below is an example for this with a delay that is easy to configure:

<details>
  <summary style="color: #e67e22; font-weight: bold;">See example</summary>

```mcfunction
# Example item
give @s minecraft:stick[minecraft:custom_data={right_click:true},minecraft:food={nutrition:0,saturation:0f,eat_seconds:2147483648f,can_always_eat:true}]
scoreboard objectives add stick.cooldown dummy
```
```json
# advancement example:stick/right_click
{
    "criteria": {
        "requirement": {
            "trigger": "minecraft:using_item",
            "conditions": {
                "item": {
                    "predicates": {
                        "minecraft:custom_data": "{right_click:true}"
                    }
                }
            }
        }
    },
    "rewards": {
        "function": "example:right_click"
    }
}

# function example:right_click
execute store result score @s stick.cooldown run time query gametime
scoreboard players add @s stick.cooldown 10
schedule function example:reset_cooldown 10t append
say Right Click!

# function example:reset_cooldown
execute store result score #reset stick.cooldown run time query gametime
execute as @a if score @s stick.cooldown = #reset stick.cooldown run advancement revoke @s only example:stick/right_click
```

</details>

This method allows you to check a right click for almost any item and you do not need to use a resourcepack to change the texture as for the CoaS / FoaS method.

##### 1.21.2+

In 1.21.2 some part of the `food` component has been separated into the `consumable` component. So we will need to change the `give` command, depending on what of the two methods you are using. The `consume_seconds` will be set to `0`, that is now possible in 1.21.2+ (if using the scoreboard method) or to `2147483647` (if using an advancement)

```
# get item
give @p stick[food={nutrition:0,saturation:0,can_always_eat:true},consumable={consume_seconds:2147483647}] 1
```

We can also add a cooldown (with the `use_cooldown` component) and use another animation instead of eating (it can be `none`, `eat`, `drink`, `block`, `bow`, `spear`, `crossbow`, `spyglass`, `toot_horn` or `brush`). Keep in mind that the item will be gone when using it. Here is a small example, detecting it using an advancement, of a nether star with the bow animation and a 5 second cooldown.

<details>
  <summary style="color: #e67e22; font-weight: bold;">See example</summary>

```mcfunction
# Setup
give @p nether_star[use_cooldown={seconds:5},food={nutrition:0,saturation:0,can_always_eat:true},consumable={consume_seconds:1,animation:"bow"},custom_data={right_click:true}] 1
```
```json
# advancement example:right_click/nether_star
{
    "criteria": {
        "requirement": {
            "trigger": "minecraft:consume_item",
            "conditions": {
                "item": {
                    "predicates": {
                        "minecraft:custom_data": "{right_click:true}"
                    }
                }
            }
        }
    },
    "rewards": {
        "function": "example:right_click"
    }
}

# function example:right_click
say used nether star
```
</details>

#### Villager method

Spawn an invisible, NoAI, Silent dummy villager in front of the player and test if the player's "talked to villager" score increases (`minecraft.custom:minecraft.talked_to_villager`, see [here](https://minecraft.wiki/Scoreboard#Criteria)).

Pros:
- Works for any item.

Cons:
- Stops players from hitting entities as well as placing and breaking blocks whilst the villager is present.  
- Zombies will path-find to the fake villager often and not the player.  
- Creates unexpected behaviour in multiplayer, as other players can also interfere with one player's fake villager; possible to abuse.  

_Parts of this post are taken and modified from [here](https://www.reddit.com/r/MinecraftCommands/comments/elnygk/item_abilities), which have been written by [u/Lemon_Lord1](https://www.reddit.com/u/Lemon_Lord1)_

### Inventory click

| 📝 Note |
|---------|
|This method works since version 1.20.5|
|This will not work for creative players.|

This method is based on checking the player's cursor slot. To do this, need to check slot `player.cursor` using the `if items` subcommand or predicates. Below is an example for running a command when holding a custom item in the cursor.

    # Setup
    give @s stick[minecraft:custom_data={in_cursor:true}]
    scoreboard objectives add hold.in_cursor dummy
    
    # Command blocks
    execute as @a[scores={hold.in_cursor=0}] if items entity @s player.cursor *[minecraft:custom_data~{in_cursor:true}] run say Cursor Command.
    execute as @a store success score @s hold.in_cursor if items entity @s player.cursor *[minecraft:custom_data~{in_cursor:true}]

## Java and Bedrock

### Bundles

| 📝 Note |
|---------|
|In Java, it is recommended to use the other methods listed above|

Right-clicking a bundle will take the first item that has. Because it will spawn an enitity, we can target it. In bedrock edition you will need [a complex method](wiki/questions/giveitembedrock) to be able to give a bundle with a renamed item inside, in this example the item is called `right_click` (so we can distinguish it from other items) and we are going to use the structure method to give the item. In Java, you can use custom data for better performance instead, but it’s recommended to use the other methods listed above.

```mcfunction
# bedrock
execute at @e[type=item,name="right_click"] run tag @p add right_click
replaceitem entity @a[tag=right_click] slot.weapon 0 air
execute at @e[tag=right_click] run structure load right_click_bundle ~ ~ ~
execute as @e[tag=rigth_click] at @s run say Right click
kill @e[type=item,name="right_click"]
tag @a[tag=right_click] remove right_click
```

Java

<details>
  <summary style="color: #e67e22; font-weight: bold;">See example</summary>

    # Example item
    give @s bundle[custom_data:{bundle_click:true},bundle_contents=[{id:"minecraft:music_disc_11",count:1,components:{"minecraft:custom_data":{right_click_bundle:true}}}]]

    # Command blocks:
    execute as @e[type=item] if contents entity @s *[custom_data:{right_click_bundle:true}] on owner run tag @s run add right_click_bundle
    execute as @e[type=item] if contents entity @s *[custom_data:{right_click_bundle:true}] run kill @s
    clear @a *[custom_data:{right_click_bundle:true}]
    execute as @a[tag=right_click_bundle] run say example
    execute as @a[tag=right_click_bundle] if items entity @s weapon.mainhand bundle[custom_data:{bundle_click:true}] run item replace entity @s weapon.mainhand with bundle[custom_data:{bundle_click:true},bundle_contents=[{id:"minecraft:music_disc_11",count:1,components:{"minecraft:custom_data":{right_click_bundle:true}}}]]
    execute as @a[tag=right_click_bundle] unless items entity @s weapon.mainhand bundle[custom_data:{bundle_click:true}] run item replace entity @s weapon.mainhand with bundle[custom_data:{bundle_click:true},bundle_contents=[{id:"minecraft:music_disc_11",count:1,components:{"minecraft:custom_data":{right_click_bundle:true}}}]]
    tag @a[tag=right_click_bundle] remove right_click_bundle
</details>