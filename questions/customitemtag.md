# Give an item a custom tag to identify it by

For example, you want to give/summon/replaceitem an item, then later easily check if a player is wearing it on their head. 

First, check out [this](https://drive.google.com/file/d/0B5GBricpOPLnSEJ2YW1ocldHVkE/view?usp=sharing&resourcekey=0-xlxvTptTpoQF-TTlFKaT_g) to get an understanding of an item's NBT structure. Note that scoreboard tags (like `@e[tag=blah]`) belong to the *dropped item entity*, and **not** the inventory item that it contains. Inventory items cannot have scoreboard tags, only the entities that contain them can (players, mules, item frames, dropped item entities, etc.). 

Normally Minecraft will remove any unknown NBT data (e.g: `data merge entity @e[limit=1] {Test:"blah"}` will be instantly removed, as that's not a valid tag for any entity), but this is not the case with NBT data in the item's `tag` tag. As Minecraft leaves data in an item's `tag` tag intact, we can put anything we want there, and later test for it.

The item's `tag` tag is also where all data from clear/replaceitem/give's `{data}` argument goes. So, if you were to give an item like this:

    /give @p stick{MyCustomTag:1b} 1

You can later test if it's in a players inventory like this:

    @a[nbt={Inventory:[{tag:{MyCustomTag:1b}}]}]

Or test if it's dropped as an item entity like this:

    /execute if entity @e[type=item,nbt={Item:{tag:{MyCustomTag:1b}}}]

The key can be any string and the value of this tag can be any NBT, so long as you test for it in the same way:

    /give @p stick{BlahBlahBlah:"string value!"} 1

[How to detect a specific item in more detail](/questions/detectitem).  

([Want to then select the player/dropped item/whatever you found with testfor (1.5-1.12)?](/questions/tagentity))