# How to make hostile mobs friendly? / disable PvP?

## Disable both hostile and PvP
This method can be used to make both, disable PvP and make hostile mobs passive.

### Teams (Java only)
In java, we can create teams, and we can configure friendly fire to make people of that team unable to attack each other. This also works for mobs. In this example we are going to make zombies passive to players.

    # in chat / load function
    team add Friendly
    team join @a friendly
    
    # tick function / command blocks
    team join @e[team=!friendly,type=zombie] friendly


### Weakness and resistance (Java and Bedrock)
If you can’t use the `/team` command (because you are using it for another thing or you are in bedrock) you can use effects. If we give resistance level 5 or higher the entity will be invulnerable to all damages except the `/kill` command. Weakness is recommended to avoid the player or mob cause knockback.

> [!NOTE]
> The entity will be invulnerable even to things as fall damage or entity craming, attacks that aren’t caused by the player

In this example we are going to make all zombies unable to attack the player for one minute

    /effect give @e[type=zombie] weakness 60 127 true

And if we want the player unable to atack the zombie

    /effect give @a weakness 60 127

> [!NOTE]
> You can still attack the player/entity if you have the sharpness enchantment, that's why we use resistance.

## Only dissable hostile
This method wont work to dissable PvP, it will only prevent the entity attacking the player.

### Helmet
Unsure if this is a bug but as show [in this Reddit post](https://new.reddit.com/r/MinecraftCommands/comments/1cuibxp/comment/l4ya7gx/) constantly using `/item` on the mob will mess up the AI and will not attack the player.

> [!NOTE]
> This does **not** work in Bedrock

    # Java syntax
    /item replace entity @e[type=skeleton] armor.head with iron_helmet

### Follow range
You can modify the follow range attribute of some mobs so they can't find you. In bedrock you can use the invisibility effect to reduce this range, or mob heads.

## Only dissable PvP
This method will not make hostile mobs passive but it will prevent players from attacking other entities or players.

### Interactions (Java only)
If it is for only one mob you can add an `interaction` entity that constantly teleports to the mob or rides it and the player will attack the interaction instead of the mob, this will make the mob unkillable with attacks

Important things to keep in mind when using this method:

* The players will **not** be able to interact with the mob, if it’s a villager you will not be able to trade with them.
* The interaction can “lag” behind if the mob moves too quickly such as when it falls if teleporting it into the mob.
* Hacked clients can override this method, and attack directly the entity with hacks such as killaura.
* Arrows and other projectiles will be able to attack the entity

### Distance attribute (Java only)
In java edition there are 2 attributes related to the range that players can interacti with the world. we can reduce this range and set it to `-4.5` to make them unable to interact with other entities.

> [!NOTE]
> This will affect right click too so they won't be able to trade with villagers for example.
