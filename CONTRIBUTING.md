# Contributing


## Codeblocks and Details

In order to get `mcfunction` syntax highlighting in an article, use code blocks with the mcfunction language defined like this

````md
```mcfunction
# Comment
say this is a command
```
````

When typing code longer than several lines, and to prevent cluttering the article with commands/:json: JSON, wrap your code into an HTML `details` tag, which will result in previewing it like this:

<details markdown="1">
  <summary style="color: #e67e22; font-weight: bold;">See commands</summary>

```mcfunction
# Command blocks
say this is a command
```
</details>

This is accomplished by typing the following code

````html
<details markdown="1">
  <summary style="color: #e67e22; font-weight: bold;">See commands</summary>

```mcfunction
# Command blocks
say this is a command
```
</details>
````
## Editing syntax highlighting

The rules of syntax highlighting are defined in `_includes/head-custom.html`. Regex definitions start at line 6 and color definitions can be found at line 185. For example, to add highlighting for a backslash:

```html
<script>
Prism.languages.mcfunction = {
  // Existing code here
  'backslash': {pattern: /\\/, greedy: true},
};
</script>

<style>
  /* existing code here */
  .token.backslash { color: #ffee11; }
</style>
```

## Table of contents & headings

Tables of contents of articles are generated automatically after the last h1 (a single `#`). For that reason, only use the h1 for the first heading (page title).

## Emoji

Emojis are supported in the wiki as <code>:emoji_name:</code>, and these are the ones available:

| Code (what you write)                  | Emoji (what will be shown) |
| -------------------------------------- | -------------------------  |
| <code>:any:</code>                     | :any:                      |
| <code>:boolean:</code>                 | :boolean:                  |
| <code>:byte:</code>                    | :byte:                     |
| <code>:byte_array:</code>              | :byte_array:               |
| <code>:double:</code>                  | :double:                   |
| <code>:float:</code>                   | :float:                    |
| <code>:int:</code>                     | :int:                      |
| <code>:int_array:</code>               | :int_array:                |
| <code>:list:</code>                    | :list:                     |
| <code>:long:</code>                    | :long:                     |
| <code>:long_array:</code>              | :long_array:               |
| <code>:short:</code>                   | :short:                    |
| <code>:string:</code>                  | :string:                   |
| <code>:struct:</code>                  | :struct:                   |
| <code>:armor_stand:</code>             | :armor_stand:              |
| <code>:aoe:</code>                     | :aoe:                      |
| <code>:area_effect_cloud:</code>       | :area_effect_cloud:        |
| <code>:fireball:</code>                | :fireball:                 |
| <code>:command_block:</code>           | :command_block:            |
| <code>:chain_command_block:</code>     | :chain_command_block:      |
| <code>:repeating_command_block:</code> | :repeating_command_block:  |
| <code>:command_blocks:</code>          | :command_blocks:           |
| <code>:folder:</code>                  | :folder:                   |
| <code>:file:</code>                    | :file:                     |
| <code>:minecraftcommands:</code>       | :minecraftcommands:        |
| <code>:json:</code>                    | :json:                     |

### Adding new emojis

To add new emojis, go to the `_includes/head-custom.html` file and add a new entry to the `emojis` object in line 112 of the file `_includes/head-custom.html`.

```js
var emojis = {
  // existing code here
  "new_name": "https://example.com/image.png"
};
```

## Article names

Keep the article names all lowercase without any separator between words, like how it's done in all existing articles.

## Redirects

You might want to add redirects to your page, like how typing `minecraftcommands.github.io/wiki/questions/item` automatically redirects to `minecraftcommands.github.io/wiki/questions/detectitem`. For that, edit the object found at line 30 in the `404.html` file.

```js
const redirects = {
  // existing code here
  "newpage" : ["commonredirect","alias","otherredirect"]
}
```

You can use this for other aliases of your page, common typos or that changing the order of the words in the article name leads to the same page.