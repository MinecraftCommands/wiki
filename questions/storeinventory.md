# Store a players inventory and return it to them at some point

This is a rather difficult thing to do that requires some advanced techniques. And due to the matter of the problem, not all details can be covered in detail in this article and are left for the reader to implement and figure out.

Thus, in your case **it might be easier to just give every player a double chest and ask them to take off their items themselves**, if you can afford/expect this level of cooperation of players in your situation.

## Bedrock

To get the items from the player, you need to kill them with the `keepinventory` gamerule switched off. Now the question is how you can store the items and give them back at a later point. (Turning on `immediateRespawn` and `showDeathMessages` off helps with making this more seemless for the player.)

For this, you need to put them somewhere in a confined space, so the items don't spill everywhere, and you also need to make sure only one player gets put into that area at a time, so the items don't get mixed up.

Same for the return part, as you only want to give the player who owns the items those items back. You also need to consider that a completely full inventory (including armor) will not be able to be picked up instantly by the player without them manually equipping their armor.

### Custom Hopper Entity

This method requires you to make a custom entity that has both an `minecraft:inventory` and `minecraft:item_hopper` component.

The `inventory_size` needs to be at least 40 slots, but it is recommended in this case to just add a few more for good measure. 

    "components": {
      "minecraft:inventory": {
        "inventory_size": 45,
        "container_type": "container"
      }
    }

Next, since you'll want to add an event that removes the `item_hopper` component and one that kills the entity (e.g. by setting its health to 0), both of these components need to be in a component group.  

    "component_groups": {
      "mcc:kill_entity": {
        "minecraft:health": {"value": 0, "max": 0}
      },
      "mcc:pickup_active": {
        "minecraft:item_hopper": {}
      }
    }

*We can't just kill the entity without removing the item hopper functionality first, as it would spew out the items on death but would pick up 1-2 items during the death animation again which would then be deleted!*

For convenience, we can create a single event that deals with both of these component groups so we only need to execute one event.

    "events": {
      "mcc:release_items": {
        "add": {"component_groups": [ "mcc:kill_entity" ]},
        "remove": {"component_groups": [ "mcc:pickup_active" ]}
      }
    }

For a better performance however I recommend to remove the item_hopper component once you're done picking up the items through a dedicated event.

You are now ready to get the items out of the player, let them be picked up by your custom entity, link them through a [scoreboard ID system](/wiki/questions/linkentity) (aka giving the entity and the player the same id score) and after the game is over, teleport the entity and the player into the same room and kill the entity (through the `/event` described above).

### `/structure`

One option that allows for an ingame solution (commands only, so without the need to make an add-on) would be to save the item entities in a structure using the `/structure save` command, and giving them back using `/structure load`.

This has the **disadvantage** that structure names cannot be dynamic, so you have to hardcode all structure names that you intend to use at some point. This also means that you're very much limited in the amount of concurrently stored player inventories to the amount of structures you hardcoded.

Please note that while an add-on isn't necessarily required for this solution, using functions is still very much recommended.

-----------

## Java

Luckily in Java we don't have to go through what is essentially a custom hopper minecart, killing the player to get their items. Instead we can use `NBT` to solve this issue for us. _Usage of functions is highly recommended and is expected for this explanation._

There are community made datapacks that do the heavy lifting for you like [PlayerDB](https://rx-modules.github.io/PlayerDB).

### Storing

- [In storage](#in-storage)
- [In marker entity](#in-marker-entity)

#### In storage

Since version 1.20.2, [macro](https://minecraft.wiki/w/Function_(Java_Edition)#Macros) have been added that allow you to insert any data into any part of the command, making storing/returning easier.

Below is an example of storing a player's inventory in storage using the [scoreboard ID system](/wiki/questions/linkentity), or you can store the UUID / nickname instead of using the scoreboard ID system. To do this you need to run the function `example:storing` as a player.

    # function example:storing
    data remove storage example:inv this
    execute store result storage example:inv this.ID int 1 run scoreboard players get @s ID
    data modify storage example:inv this.Inventory set from entity @s Inventory
    function example:storing/update with storage example:inventory this
    
    # function example:storing/update
    $execute unless data storage example:inv players[{ID:$(ID)}] run data modify storage example:inv players append {ID:$(ID)}
    $data modify storage example:inv players[{ID:$(ID)}] merge from storage example:inv this

This player data storage system will create an in storage `example:inv` object for each player in the `players` list that will look something like:

    # storage example:inv players[{ID:5}]
    {ID:5,Inventory:[{Slot:0b,id:"minecraft:stick",Count:1b}]}

However, this implementation can support any data other than player `ID` and `Inventory` and you can easily add saving any other data.

#### In marker entity

The way to do this is to store the players `Inventory` NBT somewhere safe. There are many things in the game that can store arbitrary NBT data (storage, item tag nbt, etc), and they sure are all good solutions in their own right. In this case, since we're assuming that one player is supposed to be connected to one Inventory save, without any further knowledge of the environment, we'll be using the `marker` entities ability to store arbitrary data for us. *(In case a player is bound to a certain area in a map for example, an item in a jukebox might be a better choice. In any situation, using the `storage` might be a better choice for various reasons, if you know how to properly link data in there with players.)*

So, assume we want to **store** a players inventory then. This part is the easy part, as it just takes a few commands (assuming it's executed in a function `as` the player but **not** `at` the player. Instead if possible, make sure this is executed in the spawnchunks or an otherwise ensured to be loaded chunk so the marker entities stay loaded). This also assumes you have a [scoreboard ID system](/wiki/questions/linkentity) set up to link the entity to the player.

    # summon marker
    summon marker ~ ~ ~ {Tags:["inv_store","inv_new"]}
    
    # link marker to player    
    scoreboard players operation @e[tag=inv_new] id = @s id
    
    # copy Inventory of player to marker data.Inventory
    data modify entity @e[tag=inv_new,limit=1] data.Inventory set from from entity @s Inventory
    
    # remove the new tag to get ready for the next player
    tag @e[tag=inv_new] remove inv_new

And **that's it, the entire inventory is now stored in this marker** that has been linked through a scoreboard to our player.

### Returning

But now, we **want to give it back**. This is where it becomes tricky. Choose the category from below that fits your needs the best.

- [Putting things in the original slot (from storage)](#putting-things-in-the-original-slot-from-storage)
- [Putting things in the original slot (from marker)](#putting-things-in-the-original-slot-from-marker)
- [Don't care about the slot](#dont-care-about-the-slot)
- [Put it in a chest](#put-it-in-a-chest)

#### Putting things in the original slot (from storage)

Using macros makes it very easy to return items to the player, since you can simply read the `Slot`, `id`, `Count` and `tag` tags and paste this data into the /item replace command. But there are two small difficulties:

- some slots (offhand and armor slots) cannot be directly inserted by slot number, so a separate function is needed to handle these slots.
- items may not contain a tag tag, but macros always require all the data to be inserted.

Therefore, need to create a scoreboard in which the current slot will be stored in order to run one of two functions: `example:returning/inventory` - for inventory slots (0 - 35) and `example:returning/equipment` - for offhand (-106) and armor slots (100 - 103).

For items without tags, you need to create an empty tag before running the macro for that slot.

Below is an example for versions 1.20.2 - 1.20.4. To do this need to run function `example:returning` as a player:

    # function example:load
    scoreboard objectives add Slot dummy
    
    # function example:returning
    ## Read player Scoreboard ID
    execute store result storage example:inv this.ID int 1 run scoreboard players get @s ID 
    ## Reading the selected player's data from the entire array of data of all players.
    function example:returning/read with storage example:inv this
    ## Create an empty tag if there is no tag data in the current slot.
    execute unless data storage example:inv this.Inventory[-1].tag run data modify storage example:inv this.Inventory[-1].tag set value "" 
    ## Running the function of returning items from the end of the list.
    function example:returning/item with storage example:inv this.Inventory[-1]
    
    # function example:returning/read
    $data modify storage example:inv this set from storage example:inv players[{ID:$(ID)}]
    
    # function example:returning/item
    ## Set the current slot to select slot processing
    $scoreboard players set #this Slot $(Slot)
    execute if score #this Slot matches 0..35 run function example:returning/inventory with storage example:inv this.Inventory[-1]
    execute unless score #this Slot matches 0..35 run function example:returning/equipment with storage example:inv this.Inventory[-1]
    ## After returning the current item, remove this slot from storage and start returning the next item
    data remove storage example:inv this.Inventory[-1]
    execute unless data storage example:inv this.Inventory[-1].tag run data modify storage example:inv this.Inventory[-1].tag set value "" 
    function example:returning/item with storage example:inv this.Inventory[-1]
    
    # function example:returning/inventory
    ## For inventory slots, can directly insert Slot into the /item command
    function example:returning/equipment with storage example:inv this.Inventory[-1]
    $item replace entity @s container.$(Slot) $(id)$(tag) $(Count)
    
    # function example:returning/equipment
    ## Equipment slots require converting slot number to slot name
    $execute if score #this Slot matches -106 run item replace entity @s weapon.offhand $(id)$(tag) $(Count)
    $execute if score #this Slot matches 100 run item replace entity @s armor.feet $(id)$(tag) $(Count)
    $execute if score #this Slot matches 101 run item replace entity @s armor.legs $(id)$(tag) $(Count)
    $execute if score #this Slot matches 102 run item replace entity @s armor.chest $(id)$(tag) $(Count)
    $execute if score #this Slot matches 103 run item replace entity @s armor.head $(id)$(tag) $(Count)

However, as of version 1.20.5, item tags have now been replaced with components that cannot simply be inserted into the /item command, so for this need use a loot table written inline.

Below is an example of a loot table in which you need to put the item data using a macro:

    {
      "pools": [
        {
          "rolls": 1,
          "entries": [
            {
              "type": "minecraft:item",
              "name": "$(id)",
              "functions": [
                {
                  "function": "minecraft:set_count",
                  "count": $(count)
                },
                {
                  "function": "minecraft:set_components",
                  "components": $(components)
                }
              ]
            }
          ]
        }
      ]
    }

**Note: This loot table should be inside a macro function, and not as a separate loot table file!**

Below is an updated example for version 1.20.5 without comments, since otherwise it is the same as for version 1.20.2:

    # function example:load
    scoreboard objectives add Slot dummy
    
    # function example:returning
    execute store result storage example:inv this.ID int 1 run scoreboard players get @s ID
    function example:returning/read with storage example:inv this
    execute unless data storage example:inv this.Inventory[-1].components run data modify storage example:inv this.Inventory[-1].components set value {}
    function example:returning/item with storage example:inv this.Inventory[-1]
    
    # function example:returning/read
    $data modify storage example:inv this set from storage example:inv players[{ID:$(ID)}]
    
    # function example:returning/item
    $scoreboard players set #this Slot $(Slot)
    execute if score #this Slot matches 0..35 run function example:returning/inventory with storage example:inv this.Inventory[-1]
    execute unless score #this Slot matches 0..35 run function example:returning/equipment with storage example:inv this.Inventory[-1]
    data remove storage example:inv this.Inventory[-1]
    execute unless data storage example:inv this.Inventory[-1].components run data modify storage example:inv this.Inventory[-1].components set value {} 
    function example:returning/item with storage example:inv this.Inventory[-1]
    
    # function example:returning/inventory with storage example:inv this.Inventory[-1]
    $loot replace entity @s container.$(Slot) loot {pools:[{rolls:1,entries:[{type:"minecraft:item",name:"$(id)",functions:[{function:"minecraft:set_count",count:$(count)},{function:"minecraft:set_components",components:$(components)}]}]}]}
    
    # function example:returning/equipment
    $execute if score #this Slot matches -106 run loot replace entity @s weapon.offhand loot {pools:[{rolls:1,entries:[{type:"minecraft:item",name:"$(id)",functions:[{function:"minecraft:set_count",count:$(count)},{function:"minecraft:set_components",components:$(components)}]}]}]}
    $execute if score #this Slot matches 100 run loot replace entity @s armor.feet loot {pools:[{rolls:1,entries:[{type:"minecraft:item",name:"$(id)",functions:[{function:"minecraft:set_count",count:$(count)},{function:"minecraft:set_components",components:$(components)}]}]}]}
    $execute if score #this Slot matches 101 run loot replace entity @s armor.legs loot {pools:[{rolls:1,entries:[{type:"minecraft:item",name:"$(id)",functions:[{function:"minecraft:set_count",count:$(count)},{function:"minecraft:set_components",components:$(components)}]}]}]}
    $execute if score #this Slot matches 102 run loot replace entity @s armor.chest loot {pools:[{rolls:1,entries:[{type:"minecraft:item",name:"$(id)",functions:[{function:"minecraft:set_count",count:$(count)},{function:"minecraft:set_components",components:$(components)}]}]}]}
    $execute if score #this Slot matches 103 run loot replace entity @s armor.head loot {pools:[{rolls:1,entries:[{type:"minecraft:item",name:"$(id)",functions:[{function:"minecraft:set_count",count:$(count)},{function:"minecraft:set_components",components:$(components)}]}]}]}

#### Putting things in the original slot (from marker)

This one is a little command intensive, but probably the easiest to understand. All we do is use `item replace` to return all the items back to the player, for every slot. But because we can't get the item data directly from the data storage, we first need to put it into a different container or entity inventory. So lets assume we have a chest placed at `<pos>` that we can store these items into (you can instead use an entity that has an inventory and summon it as needed).  

**This replaces all items that might still be in the players inventory.**  

Running this function `as` the marker entity and having the target player marked with the `target` tag. This assumes you have a `temp` dummy scoreboard you can use.

    # count the amount of items in the array so we know how often to repeat
    execute store result score #items temp run data get entity @s data.Inventory
    
    # if there is at least one item, start the process.
    execute if score #items temp matches 1.. positioned <pos> run function namespace:return_item
    
    # remove entity, it served its purpose. If you want to keep it around
    # you should first copy the data and work on the copy instead.
    kill @s

`return_item.mcfunction`

    # get the slot number into a scoreboard so we can use it later
    execute store result score #slot temp run data get entity @s data.Inventory[0].Slot
    # remove the Slot data so it doesn't get removed from the chest
    data remove entity @s data.Inventory[0].Slot
    # empty the chest items
    data merge block ~ ~ ~ {Items:[]}
    # copy the item data to the chest
    data modify block ~ ~ ~ Items append from entity @s data.Inventory[0]
    
    # give player the item based the slot number
    execute as @a[tag=target] run function namespace:give_correct_slot
    
    # remove item data from entity
    data remove entity @s data.Inventory[0]
    # count down the remaining slots
    scoreboard players remove #items temp 1
    # run the same function again if there are more items to process
    execute if score #items temp matches 1.. run function namespace:return_item

`give_correct_slot.mcfunction`

    # based on the previously stored slotnumber copy the item to the correct slot
    
    # offhand
    execute if score #slot temp matches -106 run item replace entity @s weapon.offhand from block ~ ~ ~ container.0
    
    # hotbar
    execute if score #slot temp matches 0 run item replace entity @s hotbar.0 from block ~ ~ ~ container.0
    execute if score #slot temp matches 1 run item replace entity @s hotbar.1 from block ~ ~ ~ container.0
    execute if score #slot temp matches 2 run item replace entity @s hotbar.2 from block ~ ~ ~ container.0
    execute if score #slot temp matches 3 run item replace entity @s hotbar.3 from block ~ ~ ~ container.0
    ...
    execute if score #slot temp matches 8 run item replace entity @s hotbar.8 from block ~ ~ ~ container.0
    
    # inv
    execute if score #slot temp matches 9 run item replace entity @s inventory.0 from block ~ ~ ~ container.0
    execute if score #slot temp matches 10 run item replace entity @s inventory.1 from block ~ ~ ~ container.0
    execute if score #slot temp matches 11 run item replace entity @s inventory.2 from block ~ ~ ~ container.0
    execute if score #slot temp matches 12 run item replace entity @s inventory.3 from block ~ ~ ~ container.0
    ...
    execute if score #slot temp matches 35 run item replace entity @s inventory.26 from block ~ ~ ~ container.0
    
    # armor
    execute if score #slot temp matches 100 run item replace entity @s armor.feet from block ~ ~ ~ container.0
    execute if score #slot temp matches 101 run item replace entity @s armor.legs from block ~ ~ ~ container.0
    execute if score #slot temp matches 102 run item replace entity @s armor.chest from block ~ ~ ~ container.0
    execute if score #slot temp matches 103 run item replace entity @s armor.head from block ~ ~ ~ container.0

#### Don't care about the slot

This means that we can just go through all the items in the array and give them back by summoning the item entity at the players feet for them to pick them up.

Thankfully we can just recursively run through all the entries/items in the array and thus summon them all in the same tick. This is assuming you're executing this `as` the linked marker entity, but `at` the player and that you have a dummy objective you can use, this example uses `temp`.

    # count the amount of items in the array so we know how often to repeat
    execute store result score #items temp run data get entity @s data.Inventory

    # if there is at least one item, start the process.
    execute if score #items temp matches 1.. run function namespace:return_items
    
    # remove entity, it served its purpose. If you want to keep it around
    # you should first copy the data and work on the copy instead.
    kill @s

`return_items.mcfunction`

    # summon a new item entity
    summon item ~ ~ ~ {Item:{id:"minecraft:stone",Count:1b},Tags:["new_item"]}
    # copy the info about the entity from the marker entity
    data modify entity @e[tag=new_item,limit=1] Item set from entity @s data.Inventory[0]
    # remove the item from the marker
    data remove entity @s data.Inventory[0]
    # remove 1 from the amount of items that we still need to process
    scoreboard players remove #items temp 1
    # remove item tag
    tag @e[tag=new_item] remove new_item
    # run the same function again if there are more items to process
    execute if score #items temp matches 1.. run function namespace:return_items

And we should be getting all our items back in a single tick. Items that do not fit in our inventory stay on the floor as item entities instead.

**However, if the player is moving or close to other players while this is supposed to happen, or if you're worried about spawning lots of item entites, use the following method instead**. It does NOT fix the issue of too many items, as the player isn't able to pick up more than 36 items, but there could be a total of up to 41 items in their inventory at the time of storing (armor and offhand). The first method will drop the item on the floor if that is the case. **The second method will not give the player anything once they reached limit due to how `/loot give` works!**

This method uses the [yellow shulkerbox loottable trick as outlined here](/wiki/questions/modifyinventory).

For this to work, we first need to get rid of the `Slot` data that is in the Inventory data (see under [explanations](#wiki_some_explanations_on_why_we_do_it_this_way) below as to why we need to do this). Then we copy the entire list of items to the shulkerbox, where it will promptly throw out all the ones that didn't fit, we use `/loot` to give its contents to the player. Then we remove the first 27 entries of the inventory (1 full shulkerbox worth) and repeat the same process with the remaining items.

So first, make sure you have the loottable from the link above added to your datapack. Also make sure you have a yellow shulkerbox placed somewhere in the world. Put the yellow shulkerboxes position into `<pos>`.

Next, run this function `as` the marker, where the target player is `@a[tag=target]`.

    # count the amount of items in the array so we know how often to repeat
    execute store result score #items temp run data get entity @s data.Inventory
    # if there is at least one item, start the process.
    execute if score #items temp matches 1.. positioned <pos> run function namespace:restore_shulker
    
    # remove entity, it served its purpose. If you want to keep it around
    # you should first copy the data and work on the copy instead.
    kill @s


`restore_shulker.mcfunction`

    # reset the shulkerbox
    data merge block ~ ~ ~ {Items:[]}
    # remove Slot data from item
    data remove entity @s data.Inventory[0].Slot
    # copy first item over to the shulkerbox
    data modify block ~ ~ ~ Items append from entity @s data.Inventory[0]
    # use /loot to give it back to the player
    loot give @a[tag=target] mine ~ ~ ~ minecraft:air{drop_contents:1b}
    # remove the first entry in the list
    data remove entity @s data.Inventory[0]
    # remove 1 from the amount of items that we still need to process
    scoreboard players remove #items temp 1
    
    # run the same function again if there are more items to process
    execute if score #items temp matches 1.. run function namespace:restore_shulker


&nbsp;


#### **Put it in a chest**

To ensure that all items that fit into a player inventory (a total of 41 items at the time of writing) are stored into a chest, we not only need a double chest, but we also need to do some NBT editing to ensure all these items actually end up in the chest. Because of how Slot numbers are checked (see [explanations](#wiki_some_explanations_on_why_we_do_it_this_way) below), coupled with the fact that chests only have 27 slots (yes, even double chests, they are just displayed as 54 slots, internally it's still 2 chests with 27 slots each), we have two choices on how to proceed:

1. Either we can hardcode all the slots again, making sure they get placed in an according slot in the chest (see the first method above) or  
2. we use a method similar to the `yellow shulker box` method (the second option of the second method), but instead of using `/loot give` we use `/loot insert`.

The first option is very similar to the option described above, with the change being that the first intermediate storage should be an entity so you can exeute positioned at the chest(s), and that you need to modify `give_correct_slot` to place the items into the slots of the chest instead of the player.

The second option is described here in more detail. See the explanation for the yellow shulker box method above.

For this we assume the two chests of the double chest are located at `~ ~ ~` and `~1 ~ ~`. Execute positioned and adjust the relative coordinates accordingly. Replace `<pos>` with the yellow shulkerbox position. Have a dummy `temp` scoreboard objective.

    # reset/empty the chests
    data merge block ~ ~ ~ {Items:[]}
    data merge block ~1 ~ ~ {Items:[]}
    # count the amount of items in the array so we know how often to repeat
    execute store result score #items temp run data get entity @s data.Inventory
    # if there is at least one item, start the process.
    execute if score #items temp matches 1.. run function namespace:into_chest
    
    # remove entity, it served its purpose. If you want to keep it around
    # you should first copy the data and work on the copy instead.
    kill @s

`into_chest.mcfunction`

    # reset the shulkerbox
    data merge block <pos> {Items:[]}
    # remove Slot data from item
    data remove entity @s data.Inventory[0].Slot
    # copy first item over to the shulkerbox
    data modify block <pos> Items append from entity @s data.Inventory[0]
    # use /loot to put it into the chest, depending on which chest it needs to be
    execute if score #items temp matches 27.. run loot insert ~ ~ ~ mine <pos> minecraft:air{drop_contents:1b}
    execute if score #items temp matches ..26 run loot insert ~1 ~ ~ mine <pos> minecraft:air{drop_contents:1b}
    # remove the first entry in the list
    data remove entity @s data.Inventory[0]
    # remove 1 from the amount of items that we still need to process
    scoreboard players remove #items temp 1
    
    # run the same function again if there are more items to process
    execute if score #items temp matches 1.. run function namespace:into_chest

### Some explanations on why we do it this way

There are various problems that we need to work around by using the above methodology.

**Player inventory cannot be edited through NBT**
If it wasn't for this, we could just store the inventory in some NBT storage and copy it back to the player at a later date. But sadly Minecraft isn't built to be able to directly manipulate the player entity so we have to do a workaround in the first place.

**Slot numbers are important**
When merging items into a chest (or other container) `Items` array, the game checks the data for some validity points, like whether it's even a proper item, whether it has a count, etc. One of the things the game checks is the `Slot` number. If this number is outside of a specific range (starting at 0, counting up however many slots a container has, so chest: 0-26, dispenser: 0-8, hopper: 0-4), this item will be discarded as well. This means, since a [player inventory](https://minecraft.wiki/images/Items_slot_number.png) has the slots 0 - 35 (inventory), plus 100 - 103 (armor) and -106 (offhand), we need to do some editing beforehand to modify / remove the `Slot` data so the items aren't discarded by the game.

**One by one**
Another trend you can see in the above solutions is that we're doing one item after another, instead of doing multiple at once. Especially with the yellow shulker box method, this might seem strange because on first thought one might think that it should be possible to just dump an entire array into the Items of the chest / box, and just letting it discard whatever doesn't fit anymore. If the items have slot numbers, that works as intended, but if the slot numbers are removed, instead of adding a new item, the slot 0 will be overwritten with each and every item, thus leading to only one item remaining.

So yes, a part of the solution could be to keep the slots around for the first 27 items, and then use the second method for the remaining items. However, the concious choice has been made in this tutorial to not mix up these two options, as it would increase the command count in this already long enough article and might lead to more confusion over the inner workings of the system.

**Slots != Index**
We often see something like `Inventory[0]` in this article. This is an array selector notation, and means that we're selecting the first item out of the array called `Inventory`. Note that any item can be in this position in the array, and it's not guaranteed to be the one that was `Slot:0b`. Only items that are actually in your inventory are saved in this list, so anything that is empty (or air I guess) will not be in this list. So if the only item in your inventory is in the last hotbar slot, it will be the only item in the list, eventhough it has `Slot:8b`.

It is possible to select an item from the array based on its Slot like this: `Inventory[{Slot:10b}]`. We could use this for the "correct slot" method, if it weren't for the fact that we can't use `/item replace` with arbitrary data but we need to use something that is guaranteed to be an item. So we can't skip the step of first putting it into a container / entity inventory first.
