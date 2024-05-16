# Check whether the player / an entity is looking at something

_This method describes how to check whether the player / an entity is **looking at something specific / predetermined**, like a specific entity or a specific position! If you're looking for a more general approach, [you'll need to use a raycast](/wiki/questions/raycast)._

## Java & Bedrock after 1.19.50

in Java and later Bedrock versions this can actually be achieved in a single command, thanks to the versatility of the `execute` command. We'll use the following subcommands to achieve our goal:

- `as @a at @s` to modify the execution entity and position.  
- `anchored eyes` to move the execution position up to the players eyes. See this related bug [MC-169665](https://bugs.mojang.com/browse/MC-169665)
- `facing <entity / coordinates>` to change the execution rotation to be facing our object / entity of desire  
- `positioned ^ ^ ^1` to move 1 block in the direction of the object  
- `anchored feet` to move the anchor back down to the players feet (due to a bug in the game which would otherwise apply the eye height modification with each position change, which we don't want, as well as to be able to check for the players existence in the last subcommand).
- `rotated as @s` to change the execution rotation back to be the same as the executing player
- `positioned ^ ^ ^-1` to move 1 block in the opposite direction of where the player is facing
- `if entity @s[distance=..0.1]` to check whether after this back and forth we've arrived roughly back at the players position. To increase / decrease the tolerance for what is considered "close enough", change the distance parameter (it needs to be between 0 and 2, because 2 basically means "you can look in the opposite direction and it's still close enough. So realistically you want to most likely stay well below 1). To calculate the exact viewing cone angle, see below.

So, to bring it all together, the full command is as follows:

    execute as @a at @s anchored eyes facing <entity / coordinates> anchored feet positioned ^ ^ ^1 rotated as @s positioned ^ ^ ^-1 if entity @s[distance=..0.1] run

Example 1: looking at the eyes of the closest cow with the tag "target":

    execute as @a at @s anchored eyes facing entity @e[type=cow,tag=target,limit=1,sort=nearest] eyes anchored feet positioned ^ ^ ^1 rotated as @s positioned ^ ^ ^-1 if entity @s[distance=..0.1] run say hello cow!

Example 2: looking at the position 10 20 30

    execute as @a at @s anchored eyes facing 10 20 30 anchored feet positioned ^ ^ ^1 rotated as @s positioned ^ ^ ^-1 if entity @s[distance=..0.1] run say hello block

## Java
There is a predicate that allows us to detect when a player is looking at an entity, it's the one used by the 3 advancements related to the spyglass.

    # function example:tick
    execute as @a[predicate=example:looking_cow] run say Hi, cow!
    

    # predicate example:looking_cow
    {
      "condition": "minecraft:entity_properties",
      "entity": "this",
      "predicate": {
        "type_specific": {
          "type": "minecraft:player",
         "looking_at": {
            "type": "minecraft:pig"
          }
        }
      }
    }

## Bedrock

### Using Commands (before 1.19.50)

In bedrock the process is the same as in Java, but due to a lack of advanced execute features, we'll need multiple commands and an entity to keep the direction and do the testing.

Also, due to how rotations work in bedrock, this tends to be somewhat inaccurate when the target is on a different y level than the player. Hopefully this will be resolved when the new execute arrives in 1.19.

This setup can only happen to one player at a time, so either use functions to make this better scaleable or experiment with executing from the players and their closest armorstands (which will still lead to some issues, but less so). Hence this will use `@p` to signify this restriction.


    # summon armorstand so we can do our check
    execute @p ~~~ summon armor_stand ~~~ none checker
    # tp armorstand to player including rotations
    execute @p ~~~ tp @e[name=checker] ~~~ ~~
    # move armorstand forward by 1 block from the players position
    # should also work if you execute as the checker instead
    execute @p ~~~ tp @e[name=checker] ^^^1
    # rotate the armorstand to face our target
    # target can be an entity or a block
    execute @e[name=checker] ~~~ tp @s ~~~ facing <target>
    # teleport the armorstand backwards from where it's looking
    # so if the player is looking the same direction, it will have moved back and forth 
    execute @e[name=checker] ~~~ tp @s ^^^-1
    # now if the as and the player are close (enough) together, the player is looking at the target
    execute @e[name=checker] ~~~ execute @p[r=0.1] ~~~ say hello there
    # remove entity
    kill @e[name=checker]

To increase / decrease the tolerance for what is considered "close enough", change the radius (`r`) selector (it needs to be between 0 and 2, because 2 basically means "you can look in the opposite direction and it's still close enough. So realistically you want to most likely stay well below 1). To calculate the exact viewing cone angle, see below.

### Using Add-Ons

To check whether the player is looking at an entity, we can use the endermans `lookat` behavior (which gets angry when looked at). It does however come with the caveat that it only tends to work around a hardcoded area around whatever the head of the entity is.

Also see [`minecraft:lookat` documentation on bedrock.dev](https://bedrock.dev/docs/stable/Entities#minecraft%3Alookat).

The lookat component then lets us fire an event which could then cause whatever we want to happen to the entity.

In this example the entity will (without cooldown) fire the `custom:ive_been_looked_at` event when a player (even in creative mode) within a radius of 20 blocks. It will not set the looker as its attack target.

    "minecraft:lookat": {
      "allow_invulnerable": true,
      "filters": {
        "test": "is_family",
        "value": "player"
      },
      "look_event": {
        "event": "custom:ive_been_looked_at"
      },
      "search_radius": 20,
      "look_cooldown": [0,0],
      "set_target": false
    }


## Calculate viewing angle

To approximate the distance/radius you want to use based on your viewing angle, you can use the following formula, where `α` is the angle that you want this method to trigger inside of, left and right of the target:

    r = 2 * sin ( α / 2 )

or, the inverse to calculate what viewing angle a certain radius / distance (`r`) value will give you

    α = sin^(-1) (r / 2) * 2

_(remember that depending on your calculator you need to convert from radians to degrees)_.

With the above calculation the example value of `r=0.1` / `distance=..0.1` leaves us with roughly a 6° angle by which we can miss the exact target in either direction and still have it considered "close enough".
