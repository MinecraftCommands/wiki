# Select an entity with *multiple* scoreboard tags

In 1.12 and below, you're unable to specify multiple selector arguments of the same key directly in a single selector. For example, `@e[tag=A,tag=B,tag=C]` ignores the first two arguments and acts the same as `@e[tag=c]`, regardless of whether the entity has tags `a` or `b`.

**In 1.13, you are able to specify multiple specify multiple tags in a selector** making this tutorial irrelevant.

## Method 1: Nested `@s` execute (1.12 and above)

With the `@s` ("self") selector and nested executes, you can target one tag on each layer, so that the command at the end is only run if all tags are present. For example, to kill someone with all tags A-E:

    execute @a[tag=A] ~ ~ ~ execute @s[tag=B] ~ ~ ~ execute @s[tag=C] ~ ~ ~ execute @s[tag=D] ~ ~ ~ kill @s[tag=E]

## Method 2: NBT tag checking

Scoreboard tags are stored in NBT as a `Tags` list. With NBT, you *can* test for multiple tags. What you can do to get around this problem then is adding a "master" tag based on their NBT data if they have all of the other tags:

    scoreboard players tag @e add HasAllTags {Tags:["A","B","C"]}

Then, you can select them with `@e[tag=HasAllTags]`.

To optimize how this performs, you can specify the least common tag in the `@e` selector, for example:

    scoreboard players tag @e[tag=A] add HasAllTags {Tags:["B","C"]}

This means that less entities will need to have their NBT checked.