# Do conditions with functions

## 1.13+

### My condition is whether or not a selector matches anything

For example, you want to execute a command when `@a[tag=TeamChange]` finds something.

Easiest way to do this is by prefixing the command with an `execute if entity`:

    execute if entity @a[tag=TeamChange,limit=1] run say test

Notice the `limit=1` in the above command, this is purely for performance reasons so the game stops after the first matching entity it found. [If you want to check whether there are an exact amount of entities found with a selector, click here](/wiki/questions/numplayers).

### My condition is whether or not another command succeeds

For this you will need to use `/execute store`.

First, set up a dummy scoreboard objective:

    scoreboard objectives add success_score dummy

We're going to use the executing entity to store the result. This can of course be substituted for a fake player or a different entity as well.

Make sure the score is initiated (not null). Easy way to do this is by adding 0:

    scoreboard players add @s success_score 0

----

Then, whenever you want to perform a conditional command:

    execute store success @s success_score if block 73 10 31 stone
    execute if score @s success_score=1 run say Stone found!
    execute if score @s success_score=0 run say Stone not found!

## 1.12

### My condition is whether or not a selector matches anything

For example, you want to execute a command when `@a[tag=TeamChange]` finds something.

Easiest way to do this is by prefixing the command with an `execute`:

    execute @a[tag=TeamChange,c=1] ~ ~ ~ say test

----

If you want to run commands without changing the executer, or run a command if the selector **fails**, you can use `/function`'s `if` or `unless` arguments. For example:

    function code:team_change if @a[tag=TeamChange]
    function code:ready unless @a[tag=!Ready]

### My condition is whether or not another command succeeds

For this you will need to use `/stats`. If you have not used `/stats` in the past, you should watch/read a tutorial (or multiple) and play around with them until you are reasonably confident in its usage and understand what they do.

First, set up a dummy scoreboard objective:

    scoreboard objectives add success_score dummy

For the entity on which you will be executing the command you want to use as a conditional, set its `SuccessCount` stat to store in its `success_score`:

    stats entity @e[tag=main] set SuccessCount @s success_score

(Note that the second selector will be evaluated **by** the entities found by the first selector. So `@s` here will target "themselves" rather than whatever's running the `/stats` command.)

Make sure the score is initiated (not null). Easy way to do this is by adding 0:

    scoreboard players add @e[tag=main] success_score 0

----

Then, whenever you want to perform a conditional command:

    testforblock 73 10 31 stone *
    execute @s[score_success_score_min=1] ~ ~ ~ say Stone found!
    execute @s[score_success_score=0] ~ ~ ~ say Stone not found!

Alternatively, if you're **not** running this function off of the entity storing its `Successcount`:

    execute @e[tag=main] ~ ~ ~ testforblock 73 10 31 stone *
    execute @e[tag=main,score_success_score_min=1] ~ ~ ~ say Stone found!
    execute @e[tag=main,score_success_score=0] ~ ~ ~ say Stone not found!

Be careful here of the order and affect that subsequent commands may have on the entity's `Successcount`. **For example, the following will not work**:

    testforblock 73 10 31 stone *
    execute @s[score_success_score=0] ~ ~ ~ say Stone not found!
    execute @s[score_success_score_min=1] ~ ~ ~ say Stone found!

That won't work because, even if the `/testforblock` succeeds, `execute @s[score_success_score=0] ...` will fail and set `success_score` to 0 before the next command runs.