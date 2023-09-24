# Item Click Detection

For item clicks we have to differentiate between leftclick and rightclick detection.

## Left-click

This one is as easily explained as it is flawed: You can only detect left clicks, if you put something in front of the player to hit. Teleport some form of entity or mob that can take damage and has a hitbox directly in the players face or even over their head, so they have no other chance but to hit that entity. You can then "detect" the clicks either using an advancement with the `player_hurt_entity` trigger ([see here](https://minecraft.wiki/Advancements/JSON_format#minecraft:player_hurt_entity)) or with a scoreboard objective of type `minecraft.custom:minecraft.damage_dealt` ([see here](https://minecraft.wiki/Scoreboard#Criteria)). This method however has many obvious flaws:  

- you're blocking the player from hitting anything else but the entity in their face  
- you're also blocking the player from building and breaking blocks  
- due to the 20Hz nature of commandblocks/functions, if the game is slightly laggy or the player is moving fast, the entity will "lag" behind, leading to unreliable detection.  

While you can use this method more reliably if this is about hitting some special mobs with a special item or something (the advancement method is really good at that), **general leftclick detection is discouraged unless you have a very controlled environment**.

## Right-click

For rightclick detection we have [a lot of different ways](https://i.imgur.com/8gKEdp1.png) (image by /u/Dieuwt), and different situations might call for different solutions. Since we cannot write a detailed guide on all these methods, we'll only describe the two most common solutions here. A rundown of the knowledge book method can be found [here](https://www.reddit.com/r/MinecraftCommands/comments/g4jxzy/simple_rightclick_detection_without_sacrificing/) (by /u/U2106_Later).

### Carrot on a stick method

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
- Has unremovable Damage NBT tag. This can be somewhat negated by making the CoaS unbreakable.

#### Make the CoaS look like any item

People tend to use a carrot on a stick and then use a resource pack to remodel them for various CustomModelData tags. This way the CoaS looks like your desired item while still providing the same rightclick functionality.

The models/item/carrot_on_a_stick.json file within the resource pack might end up looking like this:

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


### Villager method

Spawn an invisible, NoAI, Silent dummy villager in front of the player and test if the player's "talked to villager" score increases (`minecraft.custom:minecraft.talked_to_villager`, see [here](https://minecraft.wiki/Scoreboard#Criteria)).

Pros:
- Works for any item.

Cons:
- Stops players from hitting entities as well as placing and breaking blocks whilst the villager is present.  
- Zombies will path-find to the fake villager often and not the player.  
- Creates unexpected behaviour in multiplayer, as other players can also interfere with one player's fake villager; possible to abuse.  

_Parts of this post are taken and modified from [here](https://www.reddit.com/r/MinecraftCommands/comments/elnygk/item_abilities/), which have been written by /u/Lemon_Lord1_