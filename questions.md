# Common Questions

## Why is my execute command broken (bedrock)?

Because in 1.19.50, the [new execute syntax](https://learn.microsoft.com/en-us/minecraft/creator/documents/commandsnewexecute) became mandatory, so you'll need to switch to that. See also [this information on bedrock.dev](https://wiki.bedrock.dev/commands/new-execute.html).

## How do I...

### Items

[Detect a specific item (in the Inventory, in the selected slot, on the ground)?](questions/detectitem)  
[Give an item a custom tag to identify it by?](questions/customitemtag)  
[Select players with exactly X of a certain item?](questions/amountitems)  
[Detect rightclick or leftclick (on an item)?](questions/itemclick)  
[Give a special item (Bedrock)?](questions/giveitembedrock)  
[Change an item while it's in the players inventory?](questions/modifyinventory)  

### Scores

[Generate a random number?](questions/randomnumber)  
[Store an NBT value to a score, and vice versa?](questions/nbttransfer) **(1.13+ only)**  
[Check if a score is equal to, greater than, or less than another score?](questions/scorecompare)  
[Find an entity that has the same score as another entity?](questions/findsamescoreentity)  
[Summon/Teleport an entity/player at/to a position defined in a score?](questions/movetoscore)  
[Detect a change in score?](questions/changeofscore)  
[Link an entity to another entity through scoreboards?](questions/linkentity)  
[Find the player / entity with the highest score?](questions/highestscore/)   

### Players

[Activate a command *once* when a player does something (e.g: enters an area)?](questions/runonce)  
[Do something on all players in certain area(s)?](questions/areas)   
[Target a player above/below a certain Y level?](questions/heighttest)  
[Detect when a player died?](questions/playerdeaths)  
[Detect when a player kills an entity/other player?](questions/playerkills)  
[Detect a player joining (for the first time)?](questions/playerjoin)  
[Store a players inventory (and give it back later)?](questions/storeinventory)  
[Detect a player looking at something (entity / position)](questions/lookat)  

### Misc

[Do something if a command block *wasn't* successful?](questions/blockinvert)  
[Do something (e.g: kill) to the entity I just found with /execute if entity (testfor)?](questions/tagentity)  
[Add a delay to a command block?](questions/blockdelay)  
[Check if there are exactly X players matching a selector?](questions/numplayers)  
[Do conditions with functions?](questions/functionconditions)  
[Point a compass towards a player?](questions/compasstoplayer)  
[Summon an entity/projectile flying in the direction the player is looking?](questions/shootfacing)  
[Make a scoreboard ID system?](questions/linkentity)  
[Make one mob attack another mob/player?](questions/angermob)  
[Do raycasting?](questions/raycast)   
[Make a circle (of blocks / entities)?](questions/makecircle/)

## What is...

[a fake player?](questions/fakeplayer)  
[a range? / those two dots `..`?](questions/range)  
[command context?](questions/commandcontext)  


## How do I... (1.12 and below only)

[Select an entity with *multiple* scoreboard tags?](questions/multipletags)


## Still needed articles

*Found something that should be added? Maybe even want to contribute an article? Submit a pull request or create a GitHub issue.*

- Detect when a mob has died?   
- Do custom crafting (with NBT)