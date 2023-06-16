# Do something (e.g: kill) to the entity I just found with /execute if entity (testfor)

If your execute if entity **does not rely on NBT data**, you can simply move the selector into the command you want to use. For example, if you have:

    execute if entity @a[team=red]

You can also use:

    kill @a[team=red]

## Relies on NBT, 1.12 and below

If your testfor **relies on NBT data**, you will need to use either `scoreboard players set` or `scoreboard players tag` to give them a scoreboard score/tag, then select them based off of that. For example if you have:

    testfor @a {OnGround:1b}

You can give them a tag like this:

    scoreboard players tag @a add IsGrounded {OnGround:1b}

Then select them with `@a[tag=IsGrounded]`. You'll probably want to remove this tag afterwards, so that people who match the data once won't always have the tag:

    scoreboard players tag @a[tag=IsGrounded] remove IsGrounded

## Relies on NBT, 1.13 and above

In 1.13, you can specify NBT data directly in a selector, for example:

    kill @a[nbt={OnGround:1b}]