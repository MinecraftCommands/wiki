# Common Questions

## Why is my execute command broken (bedrock)?

Because in 1.19.50, the [new execute syntax](https://learn.microsoft.com/en-us/minecraft/creator/documents/commandsnewexecute) became mandatory, so you'll need to switch to that. See also [this information on bedrock.dev](https://wiki.bedrock.dev/commands/new-execute.html).

## Why is my give command broken (java)?
In 1.20.5, unstructured NBT for item stacks (tag field) was replaced with structured "components." Learn more in [this article](https://minecraft.wiki/w/Item_format/1.20.5).

## How do I...

### Items

[Detect a specific item (in the Inventory, in the selected slot, on the ground)?](/wiki/questions/detectitem)  
[Give an item a custom tag to identify it by?](/wiki/questions/customitemtag)  
[Select players with exactly X of a certain item?](/wiki/questions/amountitems)  
[Detect rightclick or leftclick (on an item)?](/wiki/questions/itemclick)  
[Give a special item (Bedrock)?](/wiki/questions/giveitembedrock)  
[Change an item while it's in the players inventory?](/wiki/questions/modifyinventory)  
[Do custom crafting?](/wiki/questions/customcrafting)  
[Make a shop? / Buy items?](/wiki/questions/shop)

### Scores

[Generate a random number?](/wiki/questions/randomnumber)  
[Store an NBT value to a score, and vice versa?](/wiki/questions/nbttransfer) **(1.13+ only)**  
[Check if a score is equal to, greater than, or less than another score?](/wiki/questions/scorecompare)  
[Find an entity that has the same score as another entity?](/wiki/questions/findsamescoreentity)  
[Summon/Teleport an entity/player at/to a position defined in a score?](/wiki/questions/movetoscore)  
[Detect a change in score?](/wiki/questions/changeofscore)  
[Link an entity to another entity through scoreboards?](/wiki/questions/linkentity)  
[Find the player / entity with the highest / lowest score?](/wiki/questions/highestscore)   

### Players

[Activate a command *once* when a player does something (e.g: enters an area)?](/wiki/questions/runonce)  
[Do something on all players in certain area(s)?](/wiki/questions/areas)   
[Target a player above/below a certain Y level?](/wiki/questions/heighttest)  
[Detect when a player died?](/wiki/questions/playerdeaths)  
[Detect when a player kills an entity/other player?](/wiki/questions/playerkills)  
[Detect a player joining (for the first time)?](/wiki/questions/playerjoin)  
[Store a players inventory (and give it back later)?](/wiki/questions/storeinventory)  
[Detect a player looking at something (entity / position)?](/wiki/questions/lookat)  

### Conditions

[Do conditions with functions?](/wiki/questions/functionconditions)  
[Do something if a command block *wasn't* successful?](/wiki/questions/blockinvert)  
[Check if there are exactly X players (or entities) matching a selector?](/wiki/questions/numplayers)  

### Entities

[Do something (e.g: kill) to the entity I just found with /execute if entity (testfor)?](/wiki/questions/tagentity)  
[Make one mob attack another mob/player?](/wiki/questions/angermob)  
[Summon an entity/projectile flying in the direction the player is looking?](/wiki/questions/shootfacing)  
[Detect when a mob has died](/wiki/questions/mobdeaths)  
[Make hostile mobs friendly / disable PvP?](/wiki/questions/hostilefriendly)  
[Setup, configure and add multiple dialogues to NPCs?](/wiki/questions/npc)

### Misc

[Add a delay to a command block?](/wiki/questions/blockdelay)  
[Point a compass towards a player/location?](/wiki/questions/compasstoplayer)  
[Make a scoreboard ID system?](/wiki/questions/linkentity)   
[Do raycasting?](/wiki/questions/raycast)   
[Make a circle (of blocks / entities)?](/wiki/questions/makecircle)  

## What is...

[a fake player?](/wiki/questions/fakeplayer)  
[a range? / those two dots `..`?](/wiki/questions/range)  
[command context?](/wiki/questions/commandcontext)  
[escaping?](/wiki/questions/escaping)  

## How do I... (1.12 and below only)

[Select an entity with *multiple* scoreboard tags?](/wiki/questions/multipletags)


## Still needed articles

*Found something that should be added? Maybe even want to contribute an article? Submit a [pull request](https://github.com/MinecraftCommands/wiki/pulls) or create a [GitHub issue](https://github.com/MinecraftCommands/wiki/issues)*
