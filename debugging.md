# Debugging

This page details information on common problems you might have with a command, and general debugging tips.

## General

* A command block [may not be running](https://bugs.mojang.com/browse/MC-86846) even when it is *Always Active* + *Unconditional* + *Repeating*. 
    * Put `/say test` in the command block to check whether it is running when you want it to run
    * Or open and close the command block. If the time on the [previous output message](http://i.imgur.com/k2rmrXS.png) is not updating, the command block is not running
    * If there is no previous output, the command may have never ran, or you have its output turned off. Turn the command block's output [on](http://i.imgur.com/s4DYa9L.png) when you are debugging
    * You can fix this by turning the command block to *Needs Redstone*, pressing *Done*, turning it back to *Always Active*, then pressing *Done* again 
* Check for double &nbsp;spaces between arguments, especially after copy pasting a part of the command. These are ignored in chat, but not elsewhere
* Don't miss out arguments. Common ones to forget are:
  * The coordinates before the NBT data in `/summon`
  * The coordinates before the command in `/execute`
  * The old block handling mode (replace, keep, etc.) before NBT data in `/setblock` or `/fill`
  * The block state/datavalue in `/setblock`, `/execute ~ ~ ~ detect` or `/fill`
* Watch out for `“smart quotes”` that word processors might auto-add, only `"normal quotes"` will work. You should use a plain plain text editor (Notepad, Notepad++, Sublime, Code), **not** a word processor or rich text editor (Microsoft Word, Wordpad, Textedit)
* Narrow down your problem as much as possible. Remove parts slowly (or build up your command slowly in the first place) until you have just the part that's causing the issue
* Macs add weird characters that are invisible in-game when the arrow keys are pressed. These will stop the command from working
* Mods/plugins (especially Essentials) may overwrite vanilla commands. Try `/minecraft:command` instead of `/command` for the vanilla implementation (e.g: `/minecraft:give`)
    * If you have any mods, try vanilla to see if the mod is causing the problem. Even mods like Optifine can cause issues

## Functions

* Make sure file extensions are not hidden ([Windows](http://i.imgur.com/FJ9x9Yg.png), [Mac](http://i.imgur.com/xDdYbXL.png)), otherwise a file that looks like `function.mcfunction` might actually be `function.mcfunction.txt`
* [Check what errors you are receiving in the game log](https://i.imgur.com/vfwl8FX.png) (this isn't chat)
* Commands cannot start with a `/` in functions (you'll get told this if you check the game log)
* Use a plain plain text editor (Notepad, Notepad++, Sublime, Code), **not** a word processor or rich text editor (Microsoft Word, Wordpad, Textedit)
* Make sure your file's encoding is UTF-8 (without BOM) - this is not the default in many programs! ([Notepad](http://i.imgur.com/R4yFjAQ.png), [Notepad++](http://i.imgur.com/8AsDJ3F.png), [Sublime](http://i.imgur.com/63rsYOB.png), [Code](http://i.imgur.com/dmOqy0y.png))
* The namespace folder is not optional, functions should **not** be directly inside `data/functions/`
* Don't forget to use `/reload` to reload the functions after making changes
* Check that you're saving to the place you think you're saving to (right world, right namespace), and running the function you intend to
* Recursive/looping functions will run `maxCommandChainLength` commands in one tick, then stop

## Selectors

* In some situations, selectors have player bias (will select players first, even if another entity is closer) or sender bias (will select the command's executer first, even if another entity is closer to the specified `x,y,z`). 
    * [See this post for more details](https://www.reddit.com/r/MinecraftCommands/comments/5url0r/selector_bias_info/ddwdek9/)
* Break down your selector into parts to see what is causing the issue. If you have `@a[r=5,tag=playing]`, try `@a[r=5]` and `@a[tag=playing]` separately
* Try using `/say` to check whether its the selector or the rest of your command that's not working 
* In 1.12 and below, you cannot include more than one argument of the same type in a selector. `@e[tag=x,tag=y,tag=z]` will act the same as just `@e[tag=z]`, ignoring the previous arguments
* Dropped item entities can have scoreboard tags, items in an inventory cannot. An item picked up then dropped will no longer have scoreboard tags applied to it
* [Selector arguments](http://minecraft.wiki/Commands#Target_selector_arguments) usually specify a maximum value, and adding `m` (or `_min` for scores) will specify a minimum value. E.G:
     * `l=9` will select anyone level 9 and **below**, `lm=9` will select anyone with level 9 and **above**
     * `score_x=5` will select anyone with score x of 5 and **below**, `score_x_min=5` will select anyone with score x of 5 and **above**
     * Specify both min and max to test for an exact value: `l=5,lm=5`
* `@a` can select dead players if none of `dx`,`dy`, `dz` or `r` are specified. No other selector can select dead players
     * Be careful with commands like: `/execute @a[tag=x] ~ ~ ~ /kill @p`
     * If any player has the tag, they'll kill themself, then kill the next nearest player (as `@p` no longer select them), then the next nearest, etc., despite only one player having the tag
     * `@s` can be used instead to select themself even if they're dead: `/execute @a[tag=x] ~ ~ ~ /kill @s`

## NBT

* Check the spelling, location, and capitalization of tags. References: [entity/world data](http://minecraft.wiki/Chunk_format), [player/item data](http://minecraft.wiki/Player.dat_format)
* Try [pca's tag checker](https://pca006132.neocities.org/pcc/nbtcheck.html)
* When testing data:
   * You must specify the tag's type. E.G: `/testfor @e {Marker:1b}` instead of just `/testfor @e {Marker:1}`
   * You must put the namespace before IDs. E.G: `/testfor @e {Item:{id:"minecraft:stone"}}` instead of just `/testfor @e {Item:{id:"stone"}}`
   * In a list, the same element can be matched multiple times. E.G: `{Motion:[0.0,1.7,2.5]}` matches `/testfor @e {Motion:[0.0,0.0,0.0]}`, as all `0.0`'s find the first `0.0`
* Item data from `/replaceitem` or `/give` is put in [the item's `tag` tag](http://minecraft.wiki/Player.dat_format#Item_structure), not directly in the item's root compound tag
* Minecraft generally won't "fix" data inside an item's `tag` tag; if you give an item with `{ench:[{id:10,lvl:1}]}`, `id` and `lvl` will stay (and need to be tested) as integers, even though they're normally shorts
* A dropped item entity stores its item data [in an `Item` compound tag](http://minecraft.wiki/Chunk_format#Items_and_XPOrbs), not directly in the entity's root compound tag
* If you need to include quotes in a string, you'll need to "escape" them by putting \ in front of them. E.G: `{Command:"/say My name is \"\"!"}`
* If you need to escape a second level, you need to escape both the quotes and the previous backslash E.G: `{Command:"/setblock ~ ~ ~ wall_sign 0 replace {Text1:\"{\\\"text\\\":\\\"hello!\\\"}\"}"}`
* [Generators are handy](https://mcstacker.bimbimma.com)
* Text editors like [Notepad++](http://i.imgur.com/7XnrEJP.png) can highlight pairs of brackets, helping you put tags in the right place and keep brackets balanced
* As of 1.12, strings containing characters that aren't `a`-`z`, `A`-`Z`, `0`-`9`, `.`, `_`, `+`, or `-` now need to be quoted. 
    * For example, `Command:say test` contains a space, so would need to be `Command:"say test"`
    * Make sure to properly escape any quotation marks contained in your command when doing this
* As of 1.12, lists can no longer have indices. 
    * For example, `Lore:[0:"aaa",1:"bbb",2:"ccc"]` would need to be `Lore:["aaa","bbb","ccc"]`
* As of 1.12, array syntax has changed. Most notable on the integer array in firework rocket colors where `Colors:[7310,31]` would now need to be `Colors:[I;7310,31]`

## Loot tables

* Check that the JSON syntax is valid with a [JSON validator](http://jsonlint.com/)
* Try out [Skylinerw's loot table evaluator](http://skylinerw.com/evaluator/)
* Double check that your loot table files are in the correct place (`world_name/data/loot_tables/namespace/optional_folders/table`)
* Double check that your command is correct (setblock needs old block handling mode, summon needs coordinates, `DeathLootTable:"namespace:optional_folders/table"` for entities and `LootTable:"namespace:optional_folders/table"` for containers)

## Resource packs

* Make sure all file names are lowercase
* The majority of textures, if not animated, need to be perfectly square (if they are animated, the height must be a multiple of the width)
* Check that all coordinates in models are between `-16` and `32`
* Check [what errors you are receiving in the game log](http://i.imgur.com/q3vhNGR.png)
* Don't forget to press `F3+T` to reload the resource pack after making changes
* Macs add invisible layers of folders to zip files. These will stop zipped resources from working
* If you're working with a zip file rather than a folder, you'll need to update the zip file after saving the files (working in a folder is easier)
* Remember that model paths have `block`/`item` (singular), whereas texture paths have `blocks`/`items` (plural)
* Check that the syntax of JSON files is valid with a [JSON validator](http://jsonlint.com/)
* Try something simple like temporarily changing the texture of an apple to verify your changes are going through

## Scoreboard objectives

* Players who have not yet had a score set do not have a score of 0, they have no score. 
    * `@a[score_x=0,score_x_min=0]` and similar will not detect players who have not had their score set yet
    * Stats also require the score to be initiated to function
    * `/scoreboard players add @e x 0` will initiate scores to 0 without affecting already set scores
* `/scoreboard players operation` requires one or both selectors to resolve to a single target
* `/stats` requires the second selector (on which the score is to be stored) to resolve to a single target