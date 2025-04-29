# What is escaping?
## Introduction to Escaping
Escaping is the process of marking special characters in a string to be interpreted literally rather than as part of a special syntax. Commands and JSON rely on precise syntax where unescaped characters can break the structure.

Escaping prevents errors and ensures that commands or JSON structures function as intended. It is essential for handling characters like quotes (`"`) or backslashes (`\`) that are part of the syntax.

Using JSON text components required escaping quotes with a backslash (`\`):

    /tellraw @a "I said \"Hi\" "

If the backslash (`\`) is removed, the quotes would be misinterpreted as the end of the string, causing a syntax error. Escaping ensures special characters like quotes or backslashes are treated as part of the message rather than command syntax.

## Combining Quotes to Avoid Escaping
When writing NBT, both types of quotes can be combined to avoid the need to escape (this is not valid for JSON): `'I said "Hi"'` = `I said "Hi"`

## Escaping backslashes
Note that this means a single backslash \ cannot be used on its own as it will attempt to escape the succeeding character, and therefore will need to be escaped itself

## Nested Strings
Escaping becomes more complex when dealing with nested strings, where one string contains another string. In such cases, you must escape both the inner and outer layers of quotes to ensure the entire structure is parsed correctly.

    give @p command_block[block_entity_data={id:"command_block",Command:"setblock ~ ~1 ~ command_block{Command:\"tellraw @a \\\\"Hi\\\\"\"}"}] 1

The Command field is enclosed in quotes (Command:"..."), so the inner quotes within this string must be escaped.

Inside the Command, the tellraw command also contains a string (tellraw @a "Hi"). To ensure "Hi" is treated literally, the quotes around it are escaped with a backslash (\").

Since this entire tellraw string is itself nested within another string, the escape characters (\") also need escaping. This is done by adding another layer of backslashes (\\\").

## Escaping in JSON Files
Escaping is also required in JSON files, such as for resource packs, advancements, or custom loot tables:

<details>
  <summary style="color: #e67e22; font-weight: bold;">See example</summary>

```json
{
  "type": "minecraft:block",
  "pools": [
    {
      "rolls": 1,
      "entries": [
        {
          "type": "minecraft:item",
          "name": "minecraft:diamond",
          "functions": [
            {
              "function": "minecraft:set_name",
              "name": {
                "text": "The \"Shiny\" Diamond",
              }
            }
          ]
        }
      ]
    }
  ]
}
```

</details>

The "text" field contains The "Shiny" Diamond. The inner quotes around Shiny are escaped as \" to be treated as part of the string. Without escaping, the parser would misinterpret the quotes and throw an error.

## Common errors

Unescaped quotes:

    "text": "This is a "broken" string"

The quotes must be escaped.

    "text": "This is a \"broken\" string"

Mismatched Quotes:

    /tellraw @a "Hi\"

This quote must not be escaped as it defines the end of the string. The backslash must be removed to be interpreted correctly.

Not nesting strings:

    give @p command_block[block_entity_data={id:"command_block",Command:"setblock ~ ~1 ~ command_block{Command:\"tellraw @a \"Hi\"\"}"}] 1

The string must be [nested](#nested-strings)