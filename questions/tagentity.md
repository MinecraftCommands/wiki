# Do something (e.g: kill) to the entity I just found with /execute if entity (testfor)

  - [Relies on NBT, 1.12 and below](#relies-on-nbt-112-and-below)
  - [Relies on NBT, 1.13 and above](#relies-on-nbt-113-and-above)

If your `execute if entity` **does not rely on NBT data**, you can simply move the selector into the command you want to use. For example, if you have:

```mcfunction
execute if entity @a[team=red]
```

You can also use:

```mcfunction
kill @a[team=red]
```

## Relies on NBT, 1.12 and below

If your testfor **relies on NBT data**, you will need to use either `scoreboard players set` or `scoreboard players tag` to give them a scoreboard score/tag, then select them based off of that. For example if you have:

```mcfunction
testfor @a {OnGround:1b}
```

You can give them a tag like this:

```mcfunction
scoreboard players tag @a add IsGrounded {OnGround:1b}
```

Then select them with `@a[tag=IsGrounded]`. You'll probably want to remove this tag afterwards, so that people who match the data once won't always have the tag:

```mcfunction
scoreboard players tag @a[tag=IsGrounded] remove IsGrounded
```

## Relies on NBT, 1.13 and above

In 1.13, you can specify NBT data directly in a selector, for example:

```mcfunction
kill @a[nbt={OnGround:1b}]
```