# How to give a special item (Bedrock)

This article is for **Bedrock only**, as Java can just use NBT to give the special items directly.

Using `/give` on bedrock only works for "basic" items, without any alterations like change of names or enchantments.

Thus, a different method needs to be followed, of which there are two (three) common ones. The last one in the list is probably the best one currently available.

### Physical chest

Either use step 4 or step 6, reasons and problems see below

1. put item into a container (e.g. chest) of your choice in a location out of sight
2. clone container to a 1x1 hole
3. use /setblock or /fill to destroy the container, making it drop its contents (and itself)
4. **either** `/kill` the dropped container item
5. tp the dropped items to the player
6. **or** if you didn't do #4, you can /clear the container from the players inventory instead

Step 4: requires you to either rename the container or hope the player is playing in english, because @e[type=item,name="Chest"] only works in english. Alternatively you can change the language files for every language so all and every chest is called "Chest" in every language.  
Step 6: Has the advantage that you don't need to fiddle with the language files or renamed containers, but has the problem that the item may not be picked up instantly thus making it harder to set up properly.

### Structure block

Instead of storing the item in a chest, you store it in a structure block by saving a 1x1 structure that consists of just air and the item entities. So the step by step guide is:

1. throw item(s) on top of a structure block, save the structure with entities (probably as a 1x1x1 sized structure).
2. change the structure block to load mode
3. whenever you need it, setblock a redstone block next to the structure block (and remove it again a tick later)
4. teleport the item entities to the player

This has the advantage that you don't have to remove the container entity after you broke it and that the items don't have a pickup delay anymore.  
It has the disadvantage that it can take a tick or two for the structure to be loaded properly and thus for the item entity to appear, which means you need a delay in your functions.

#### Structure command

**_Probably the best method currently available_**

Same as the structure block method, but instead of loading it with a structureblock, you load it using the [/structure](https://minecraft.wiki/Commands/structure) command. It has the advantage over the block method that you don't need to worry about the whole redstone block setting and the issues that come with that, you can just load the structure directly at the player in question and be done with it.

Make sure to save the structure with a structure void block inside, or the air might override whatever block the player is currently standing on.