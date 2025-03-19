# What is command context?

All commands exist in a __context__.  

This context is first set when the command is run and consists of multiple parts that all describe the "situation" that a command is run in.   

- the execution position (x / y / z and dimension)  
- the executing entity  
- the execution rotation  
- (the anchor)  

All of which can be modified individually with the 1.13+ Java as well as 1.19.1+ Bedrock execute command.

See [the Minecraft wiki on the execute command](https://minecraft.wiki/wiki/Commands/execute) for which subcommands modify which context.

## Defaults

When a command is run, it will get some default values depending on how it is run.

- A command **run from chat** is executed `as` and `at` (so positioned and rotated as) the player who runs it.
- A command **run from a commandblock** is executed with the position set to the center of the commandblock in all three axes. It is always rotated `0 0`, so facing straight forward in south (positive z) direction. _There is no executing entity_.  
- A command **run from a function** can have various values, as functions themselves keep context. See below.
   - A scheduled command does not keep context
   - Tick and load functions run at world spawn
- A command run from the server console is executed at world spawn.
- A command run from an [**NPC**](/wiki/questions/npc) <sup>\[Bedrock Edition Only\] </sup> is run `as` and `at` the `NPC`

## Context in Functions

Functions _keep the context_ that they're run in.  

So a function run in chat will have the same defaults as any other command run from a player in chat. The same is true for a function command run from a commandblock, it will have the same context as any command run from a commandblock would.  

A command run from either the `#minecraft:load` or `#minecraft:tick` [function tags](https://minecraft.wiki/wiki/Tag#Function_tags) will run positioned at world spawn (at the lowest end of the block, unlike the commandblock), rotated `0 0` and without an executing entity.

This allows a lot of [optimization](/wiki/optimising) since you can use a selector once, running the function as that entity and then refer to that entity as `@s` for the rest of the function. It also allows for [entityless raycasting](/wiki/questions/raycast#wiki_without_an_entity) as the position and rotation are preserved between function calls.

You can find a more in depth explanation on [The Minecraft Wiki](https://minecraft.wiki/w/Command_context)