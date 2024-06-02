# Give an item a custom tag to identify it by

* [Java](#Java)
* [Bedrock](#Bedrock)

## Java

For example, you want to give/summon/item an item, then later easily check if a player is wearing it on their head. 

First, check out [this](https://drive.google.com/file/d/0B5GBricpOPLnSEJ2YW1ocldHVkE/view?usp=sharing&resourcekey=0-xlxvTptTpoQF-TTlFKaT_g) to get an understanding of an item's NBT structure. Note that scoreboard tags (like `@e[tag=blah]`) belong to the *dropped item entity*, and **not** the inventory item that it contains. Inventory items cannot have scoreboard tags, only the entities that contain them can (players, mules, item frames, dropped item entities, etc.). 

Normally Minecraft will remove any unknown NBT data (e.g: `data merge entity @e[limit=1] {Test:"blah"}` will be instantly removed, as that's not a valid tag for any entity), but this is not the case with NBT data in the item's `tag` tag (1.20.4 and below) or `custom_data` component (1.20.5 and above). As Minecraft leaves data in an item's `tag` tag / `custom_data` component intact, we can put anything we want there, and later test for it.

**Since version 1.20.5, unstructured NBT data attached to stacks of items (tag field) has been replaced with structured 'components'. Therefore, now you need to use the `minecraft:custom_data` component to give an item with custom data [\[See changelog\]](https://minecraft.wiki/w/Java_Edition_1.20.5#Command_format_2).**

Below is the general syntax for the give item with some data for the new and previous versions:

    # 1.20.5 and above
    give @s <item>[<component>=<data>]
    
    # 1.20.4 and below
    give @s <item>{<nbt_data>}

Now you can’t immediately specify your custom data, but you must first specify the `minecraft:custom_data` component (in the /give command you can omit namespace `minecraft:`) and specify your data inside this component.

The item's `tag` tag / `custom_data` component is also where all data from clear/item/give's `{data}` (1.20.4 and below) or `[<component>=<data>]` (1.20.5 and above) argument goes.

So, if you were to `give` an item like this:

    # 1.20.5 and above
    give @s minecraft:stick[minecraft:custom_data={my_custom_tag:true}]

    # 1.20.4 and below
    give @s minecraft:stick{my_custom_tag:true}

Or like this if you want to `summon` item:

    # 1.20.5 and above
    summon item ~ ~ ~ {Item:{id:"minecraft:stick",components:{"minecraft:custom_data":{my_custom_tag:true}}}}

    # 1.20.4 and below
    summon item ~ ~ ~ {Item:{id:"minecraft:stick",Count:1b,tag:{my_custom_tag:true}}}

You can later test if it's in a players inventory like this:

    # 1.20.5 and above
    @a[nbt={Inventory:[{components:{"minecraft:custom_data":{my_custom_tag:true}}}]}]

    # 1.20.4 and below
    @a[nbt={Inventory:[{tag:{my_custom_tag:true}}]}]
    
Or test if it's dropped as an item entity like this:

    # 1.20.5 and above
    execute as @e[type=item] if items entity @s contents *[custom_data~{my_custom_tag:true}]
    execute if entity @e[type=item,nbt={Item:{components:{"minecraft:custom_data":{my_custom_tag:true}}}}]

    # 1.20.4 and below
    execute if entity @e[type=item,nbt={Item:{tag:{my_custom_tag:true}}}]

[How to detect a specific item in more detail](/wiki/questions/detectitem).

The key can be any string and the value of this tag can be any [NBT](https://minecraft.wiki/w/NBT_format), so long as you test for it in the same way:

    give @s stick[custom_data={BlahBlahBlah:"string value!"}]
    give @s stick{BlahBlahBlah:"string value!"}

([Want to then select the player/dropped item/whatever you found with testfor (1.5-1.12)?](/wiki/questions/tagentity))

If you use a datapack, you can also create items with custom data using a loot table or recipes (1.20.5+).

When using a loot table since version 1.20.5, you need to use the `minecraft:set_custom_data` or `minecraft:set_components` [loot function](https://minecraft.wiki/w/Item_modifier) instead of `minecraft:set_nbt` to create an item with custom data.

Below are examples of loot tables for different versions. **For version 1.20.5+, two ways to set custom data are shown, but you should only use one.**

```
# 1.20.5 and above
{
  "pools": [
    {
      "rolls": 1,
      "entries": [
        {
          "type": "minecraft:item",
          "name": "minecraft:stick",
          "functions": [
            {
              "function": "minecraft:set_custom_data",
              "tag": "{my_custom_tag:true}"
            },
            {
              "function": "minecraft:set_components",
              "components": {
                "minecraft:custom_data": {"my_custom_tag": true}
              }
            }
          ]
        }
      ]
    }
  ]
}

# 1.20.4 and below
{
  "pools": [
    {
      "rolls": 1,
      "entries": [
        {
          "type": "minecraft:item",
          "name": "minecraft:stick",
          "functions": [
            {
              "function": "minecraft:set_nbt",
              "tag": "{my_custom_tag:true}"
            }
          ]
        }
      ]
    }
  ]
}
```

The new `set_custom_data` loot function is exactly the same as the `set_nbt` function, but sets data only to the `custom_data` component. But using `set_components` for set `custom_data` is different because it cannot be represented as a string with NBT data, but it must be a JSON object and follow [JSON formatting](https://minecraft.wiki/w/JSON). Thus, any text must be escaped, and numeric values must be specified without a variable type.

Therefore, when receiving an item, this data will be converted to NBT format according to approximately the following rules:

* any string will remain a string
* any integer will be converted to a numeric variable with the smallest possible size<sup>[1]</sup>.
* Any non-integer number will be converted to double variable type<sup>[2]</sup>.
* The list of numeric variables will be converted to an array of numeric values if possible<sup>[3]</sup>.

\[1\] A number from -128 to 127 will be converted to a **byte** value. The number between -32768 to 32767 is a **short** variable type, etc.

\[2\] A value that cannot be rounded to the nearest integer without loss of precision.

\[3\] Only if all values in the list are of the same numeric variable type after conversion. Otherwise, a list of objects with an empty variable and value will be created [\[bug?\]](https://i.imgur.com/ZXndsgB.png).

It follows from this that when creating a loot table / recipe with custom data, it is worth keeping in mind that unexpected changes in values are possible.

For example, the crafting recipe below (1.20.5+), which will give an item with the custom tag `my_custom_stick:1b`, but not `my_custom_stick:1`, as indicated in the recipe due to the implicit conversion.

```
{
  "type": "minecraft:crafting_shapeless",
  "ingredients": [
    {
      "item": "minecraft:stick"
    }
  ],
  "result": {
    "id": "minecraft:stick",
    "components": {
      "minecraft:custom_data": {"my_custom_stick": 1}
    }
  }
}
```
## Bedrock

In bedrock we will need use a workarround, because we can't use custom tags.

### Item data
To get a item with a specified data, you can use a number between `2,147,483,647` and `-2,147,483,648`.

    give <target> <item> <amount> <data>
So for example:

    give @s stick 1 5

This stick will have a data of 5, that we can detect it with the `hasitem` argument like this:

    /effect @a[hasitem={item=stick,data=5}] speed

### Name with color codes
As proposed by [u/V1beRater](https://www.reddit.com/user/V1beRater/) in [this reddit post](https://www.reddit.com/r/MinecraftCommands/comments/xzbj5t/comment/irlhawd/). We can use color codes to change the name of an item of an apple to, for example, `§r§fApple`, wich is indistinguishable from a normal apple name, but you can detect if it's dropped with this command:

    kill @e[type=item, name="§r§fApple"]

