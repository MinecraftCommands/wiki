# Debugging

This page details information on common problems you might have with a command, and general debugging tips.

## General

* A command block [may not be running](https://bugs.mojang.com/browse/MC-86846) even when it is *Always Active* + *Unconditional* + *Repeating*.
    * Put `/say test` in the command block to check whether it is running when you want it to run. Make sure you have chat activated in the client options
    * Or open and close the command block. If the time on the [previous output message](http://i.imgur.com/k2rmrXS.png) is not updating, the command block is not running
    * If there is no previous output, the command may have never ran, or you have its output turned off. Turn the command block's output [on](http://i.imgur.com/s4DYa9L.png) when you are debugging
    * You can fix this by turning the command block to *Needs Redstone*, pressing *Done*, turning it back to *Always Active*, then pressing *Done* again 
* Check for double &nbsp;spaces between arguments, especially after copy pasting a part of the command. These are ignored in chat, but not elsewhere
* Check for a space at the end of the command, command should **not** end with a space
* Don't miss out arguments. Common ones to forget are:
  * The coordinates before the NBT data in `/summon`
  * The coordinates before the command in `/execute`
  * The old block handling mode (replace, keep, etc.) before NBT data in `/setblock` or `/fill`
  * The block state/datavalue in `/setblock`, `/execute if block` (or in pre-1.13 `/execute ~ ~ ~ detect`) or `/fill`
* Watch out for `“smart quotes”` that word processors might auto-add, only `"normal quotes"` will work. You should use a plain plain text editor (Notepad, Notepad++, Sublime, Code), **not** a word processor or rich text editor (Microsoft Word, Wordpad, Textedit)
* Narrow down your problem as much as possible. Remove parts slowly (or build up your command slowly in the first place) until you have just the part that's causing the issue
* Macs add weird characters that are invisible in-game when the arrow keys are pressed. These will stop the command from working
* Mods/plugins (especially Essentials) may overwrite vanilla commands. To avoid this you can use one of the two following methods:
  * `/minecraft:command` instead of `/command` for the vanilla implementation (e.g: `/minecraft:give`)
  * `/execute run command` instead of `/command`, this is better because it works in vanilla servers too
    * If you have any mods, try vanilla to see if the mod is causing the problem. Even mods like Optifine can cause issues
    * Make sure you don't have a mod that dissables command blocks
* Make sure command blocks are enabled in server.propieties
* When using tutorials or [online generators](/wiki/resources) make sure that are for the correct edition (java or bedrock) and version (1.12, 1.17, 1.20)
* Check if the command block is in a loaded chunk, you can use the [`/forceload`](https://minecraft.wiki/w/Commands/forceload) command in Java or the [`/tickingarea`](https://minecraft.wiki/w/Commands/tickingarea) command in Bedrock to force a chunk to be always loaded
* Maybe you set the command to conditional accidentaly and it should be uncoditional, double check that
* The command block must be `always active` or have redstone powering in order for it to run the command
* Make sure to capitalize the correct leters in the command, for example `/Say` will not work but `/say` will do (in bedrock edition works different, as you can capitalize commands)
  * Same goes for scoreboard values, if you capitalized it when creating it, it should be capitalized when you use it
* The `commandModificationBlockLimit` gamerule (defaults to 32768) specifies the limit of blocks that can be selected with the `/fill`, `/fillbiome` and `/clone` commands

## Functions

* Make sure file extensions are not hidden ([Windows](http://i.imgur.com/FJ9x9Yg.png), [Mac](http://i.imgur.com/xDdYbXL.png)), otherwise a file that looks like `function.mcfunction` might actually be `function.mcfunction.txt`
* [Check what errors you are receiving in the game log](https://imgur.com/a/HWQUUjX) (this isn't chat). It can be enabled on the launcher settings.
* Commands cannot start with a `/` in functions (you'll get told this if you check the game log)
* Macros lines **must** start with `$` and the values are specified using `$(example)`
  * Macro commands **must** contain at least one insert macro
  * Inserting from a list/array or NBT path is not allowed
* Use a plain plain text editor (Notepad, Notepad++, Sublime, Code), **not** a word processor or rich text editor (Microsoft Word, Wordpad, Textedit)
* Make sure your file's encoding is UTF-8 (without BOM) - this is not the default in many programs! ([Notepad](http://i.imgur.com/R4yFjAQ.png), [Notepad++](http://i.imgur.com/8AsDJ3F.png), [Sublime](http://i.imgur.com/63rsYOB.png), [Code](http://i.imgur.com/dmOqy0y.png))
* The namespace folder is not optional, functions should **not** be directly inside `data/functions/`, they must be in `data/<namespace>/functions/`
* Don't forget to save changes if editing directly the datapack (with programs like VScode), normaly `ctrl+s` (Windows/Lunix) or `cmd+s` (Mac)
* Don't forget to use `/reload` to reload the functions after making changes
  * Make sure you have the datapack enabled. You can enable the datapack by typing `/datapack enable "<datapack name>"`
  * When edition world generation you need to leave and rejoin the world in order to save changes (or restart the server).
* Check that you're saving to the place you think you're saving to (right world, right namespace), and running the function you intend to
* Recursive/looping functions will run `maxCommandChainLength` commands in one tick, then stop, the default value of this gamerule is `65536`
* Make sure the tick/load function tag is specifing the correct function
* In snapshot [24w21b](https://www.minecraft.net/en-us/article/minecraft-snapshot-24w21a) some registry types that used legacy datapack directory names (based on plural name of element) have been renamed to match registry name. Affected directories:
  * `structures` -> `structure`
  * `advancements` -> `advancement`
  * `recipes` -> `recipe`
  * `loot_tables` -> `loot_table`
  * `predicates` -> `predicate`
  * `item_modifiers` -> `item_modifier`
  * `functions` -> `function`
  * `tags/functions` -> `tags/function`

## Selectors

* In some situations, selectors have player bias (will select players first, even if another entity is closer) or sender bias (will select the command's executer first, even if another entity is closer to the specified `x,y,z`). 
    * [See this post for more details](https://www.reddit.com/r/MinecraftCommands/comments/5url0r/selector_bias_info/ddwdek9/)
* Break down your selector into parts to see what is causing the issue. If you have `@e[type=zombie,tag=playing]`, try `@e[type=zombie]` and `@e[tag=playing]` separately
* Try using `/say` to check whether its the selector or the rest of your command that's not working, make sure you have chat enabled in client options
* In 1.12 and below, you cannot include more than one argument of the same type in a selector. `@e[tag=x,tag=y,tag=z]` will act the same as just `@e[tag=z]`, ignoring the previous arguments. You can find a solution to this problem [cheking nbt](/wiki/questions/multipletags)
* Dropped item entities can have scoreboard values and tags, items in an inventory cannot. An item picked up then dropped will no longer have scoreboard values and will lose all tags applied to it
* [Selector arguments](http://minecraft.wiki/Commands#Target_selector_arguments) usually specify a maximum value, and adding `m` (or `_min` for scores) will specify a minimum value. E.G (in older versions):
     * `l=9` will select anyone level 9 and **below**, `lm=9` will select anyone with level 9 and **above**
     * `score_x=5` will select anyone with score x of 5 and **below**, `score_x_min=5` will select anyone with score x of 5 and **above**
     * Specify both min and max to test for an exact value: `l=5,lm=5`
     * This is not the case in newer versions since it uses [ranges](/wiki/questions/ranges). E.G: `@a[scores={x=5..10}]`
* `@a` can select dead players if none of `dx`,`dy`, `dz` or `r` are specified. No other selector can select dead players (for example `@e[type=player]` will only select living players)
     * Be careful with commands like: `/execute at @a[tag=x] run kill @p`
     * If any player has the tag, they'll kill themself, then kill the next nearest player (as `@p` no longer select them), then the next nearest, etc., despite only one player having the tag
     * `@s` can be used instead to select themself even if they're dead: `/execute as @a[tag=x] run kill @s`
* `as` and `at` have a diference in [command context](/wiki/questions/commandcontext). `as` change the executor entity and `at` changes the position.
     * Scheduled functions will lose the context
* Some commands like `/data` or `/damage` in Java edition can only select one target
  * The command `/data merge entity @e[type=armor_stand] {Invisible:1b}` will **not** work
    * You can use `/execute` to select more than one entity, so `/execute as @e[type=armor_stand] run data merge entity @s {Invisible:1b}` will work and modify the data of every loaded `armor_stand`
  * You can add a `limit`, so `/data merge entity @e[type=armor_stand,limit=1,sort=nearset] {Invisible:1b}` will work and will select the nearest `armor_stand`
    * In 1.21 you can use `@n` to select the nearest entity instead of `@e[limit=1,sort=nearest]`
* `sort` has no effect if `limit` is not specified
  * The selector `@e[sort=nearest]` is the same as `@e`, you will need to use `@e[limit=1,sort=nearest]` (or `@n` in 1.21+) in order to select the nearest entity


## NBT

* Check the spelling, location, and capitalization of tags. References: [entity/world data](http://minecraft.wiki/Chunk_format), [player/item data](http://minecraft.wiki/Player.dat_format)
* Try [pca's tag checker](https://pca006132.neocities.org/pcc/nbtcheck.html)
* When testing data:
   * You must specify the tag's type. E.G: `/execute if entity @e[nbt={Marker:1b}]` instead of just `/execute if entity @e[nbt={Marker:1}]`
   * You must put the namespace before IDs. E.G: `/execute if entity @e[nbt={Item:{id:"minecraft:stone"}}]` instead of just `/execute if entity @e[nbt={Item:{id:"stone"}}]`
   * In a list, the same element can be matched multiple times. E.G: `{Motion:[0.0,1.7,2.5]}` matches `/testfor @e {Motion:[0.0,0.0,0.0]}`, as all `0.0`'s find the first `0.0`
* Item data from `/item` or `/give` is put in [the item's `tag` tag](http://minecraft.wiki/Player.dat_format#Item_structure), not directly in the item's root compound tag
* Minecraft generally won't "fix" data inside an item's `tag` tag; if you give an item with `{ench:[{id:10,lvl:1}]}`, `id` and `lvl` will stay (and need to be tested) as integers, even though they're normally shorts
* A dropped item entity stores its item data [in an `Item` compound tag](http://minecraft.wiki/Chunk_format#Items_and_XPOrbs), not directly in the entity's root compound tag
* If you need to include quotes in a string, you'll need to "escape" them by putting \ in front of them. E.G: `{Command:"/say My name is \"\"!"}`
* If you need to escape a second level, you need to escape both the quotes and the previous backslash E.G: `{Command:"/setblock ~ ~ ~ wall_sign 0 replace {Text1:\"{\\\"text\\\":\\\"hello!\\\"}\"}"}`
* [Generators are handy](https://mcstacker.com) you can find some in the [Java resources page](/wiki/resources) or in the [Bedrock resources page](/wiki/bcresources)
* Text editors like [Notepad++](http://i.imgur.com/7XnrEJP.png) can highlight pairs of brackets, helping you put tags in the right place and keep brackets balanced
* As of 1.12, strings containing characters that aren't `a`-`z`, `A`-`Z`, `0`-`9`, `.`, `_`, `+`, or `-` now need to be quoted. 
    * For example, `Command:say test` contains a space, so would need to be `Command:"say test"`
    * Make sure to properly escape any quotation marks contained in your command when doing this
* As of 1.12, lists can no longer have indices. 
    * For example, `Lore:[0:"aaa",1:"bbb",2:"ccc"]` would need to be `Lore:["aaa","bbb","ccc"]`
* As of 1.12, array syntax has changed. Most notable on the integer array in firework rocket colors where `Colors:[7310,31]` would now need to be `Colors:[I;7310,31]`
* You can **not** [edit player data](https://www.reddit.com/r/MinecraftCommands/comments/9ty42s/data_modify_is_irritating_me_today/e91rwc7/?context=3), unless when using a mod, for example [Edit Player NBT](https://modrinth.com/mod/edit-player-nbt).

## Loot tables

* Check that the JSON syntax is valid with a [JSON validator](http://jsonlint.com/)
* Try out [Skylinerw's loot table evaluator](http://skylinerw.com/evaluator/)
* Double check that your loot table files are in the correct place (`world_name/data/loot_tables/namespace/optional_folders/table`)
* Double check that your command is correct (setblock needs old block handling mode, summon needs coordinates, `DeathLootTable:"namespace:optional_folders/table"` for entities and `LootTable:"namespace:optional_folders/table"` for containers)

## Resource packs

* Make sure all file names are lowercase
* The majority of textures, if not animated, need to be perfectly square (if they are animated, the height must be a multiple of the width)
* Check that all coordinates in models are between `-16` and `32`
* Check [what errors you are receiving in the game log](https://imgur.com/a/HWQUUjX)
* Don't forget to press `F3+T` to reload the resource pack after making changes
* Check if you have the resource pack enabled
* Macs add invisible layers of folders to zip files. These will stop zipped resources from working
* If you're working with a zip file rather than a folder, you'll need to update the zip file after saving the files (working in a folder is easier)
* Remember that model paths have `block`/`item` (singular), whereas texture paths have `blocks`/`items` (plural)
* Check that the syntax of JSON files is valid with a [JSON validator](http://jsonlint.com/)
* Try something simple like temporarily changing the texture of an apple to verify your changes are going through
* Text display that displays translation key will not only require reloading the resource pack when changing the translation, it **will** require rejoining the world in order to see the changes.
* Resource packs can override others
* Some resource packs will **not** work with client side optimizaton mods such as Sodium, for example vanilla tweaks fullbright.

## Scoreboard objectives

* Players who have not yet had a score set do not have a score of 0, they have no score. 
    * `@a[scores={x=-2147483647..}]` and similar will not detect players who have not had their score set yet
    * Stats also require the score to be initiated to function
    * `/scoreboard players add @e x 0` will initiate scores to 0 without affecting already set scores
    * You can check if a player has a score set or no with the command `/execute as @a unless score @s x = @s x run ...`
* `/scoreboard players operation` requires one or both selectors to resolve to a single target.
* The minimum and maximum value of a scoreboard is ±2,147,483,647
* There are scoreboards criterias that can **not** be edited such as hunger or health.
* In order to people to use the `/trigger` command, the scoreboard **must** be enabled for that player, you can enable a trigger with this command: `/scoreboard players enable <player_selector> <objective>`.
  * If you reset the scoreboard the player will no longer has the objective enabled
  * When you use the `/trigger` command that scoreboard no loner is enabled
* Scoreboard values are linked to the username **not** to the player `UUID`
