# Find player / entity with the highest score

> [!NOTE]
> This only works with online players / loaded entities. If you want to accomodate offline players / unloaded entities, you'll need a much more complicated system!

Thanks to `scoreboard player operations` this is a fairly easy question to answer, as the `>` operator will ensure the left score is at least as high as the right score.

Thus the way to do this is as follows (assuming you're trying to find the highest score in an objective named `score`).

First you set up a [fake player](/wiki/questions/fakeplayer) with the smallest possible score you can reasonably have, or alternatively you can use the smallest possible number (`-2147483648`).

    scoreboard players set #max score -2147483648

then you execute as all the entities you want to compare, running the scoreboard operation as all of them. Here we'll use all players in the example.

    execute as @a run scoreboard players operation #max score > @s score

And now you can find whoever has the same score as the fake player ([using Method #2 from here](/wiki/questions/findsamescoreentity/)) to name your winner.

    execute as @a if score @s score = #max score run I have the highest score!
