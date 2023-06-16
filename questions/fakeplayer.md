# What is a fake player

When people talk about "using a fake player" on the scoreboard, they're talking about just writing down a name of a fictional playername instead of using a selector. This has the advantage that it is very fast and convenient to store and use scoreboard values.

    scoreboard players set total points 100
    scoreboard players set 2 constant 2

It's generally used for storing constants (e.g. when you need to divide a number by 20, you need to store that somewhere), global variables (e.g. for settings in a datapack or map), or temporary variables, but can be used whenever it seems helpful.

They can be used like any other selector, but only make sense when dealing with scoreboards, as those don't require the selected player to be online (which is impossible with a fake player). It is highly favored over storing values on a persistent entity, as for that entity you need an `@e` selector which is significantly more performance hungry than using a fake player. Additionally a single objective can hold an near infinite number of fake player scores, while only one score per real entity can be stored.

## Special characters

It is recommended to use special characters inside the fake player names to prevent conflicts with real players (see [things to look out for](#things_to_look_out_for)). While you cannot use spaces, you can use a lot more characters that normally wouldn't be possible to be used in normal playernames like `.#$%/*!`. The three most common ones are 

- `.` generally used to create a sort of namespace (e.g. `homes.x`, `homes.y`)  
- `$` tends to be used at the start of the fake name. This likely stems from other programming languages where this character symbolises a variable.  
- `#` also used at the start, as it has a **special feature**: If it's the first character in the name, it is not displayed on the sidebar when the objective is displayed there.

## Things to look out for

When you use this method, if you're using something that could reasonably be an actual player name (like [`Max`](https://namemc.com/profile/Max.1)), and that player happens to be online at that time, you might run into conflicts. As such it is recommended to use those special characters to make sure no conflicts are possible or use seperate scoreboard objectives for fake player scores and real player scores.