# Nbt path


## Small introduction

The Named Binary Tag (NBT) is a tree data structure used by Minecraft in many save files to store arbitrary data. Stringified Named Binary Tag (SNBT) format is the accessible version of it as strings.

Example of SNBT:
```snbt
{
  key1: 981,
  'key2': 'value',
  "key3": {
    subkey1: 1b,
    "subkey2": "another value"
  }
}
```

We are not going to explain in full depth what NBT is, since this focuses on NBT paths, more information can be found on the wiki: https://minecraft.wiki/w/NBT_format.

## Data paths

While `data merge` allows you to modify in SNBT, using `data modify` requires a path. Other commands, like `execute if data` and `execute store` also uses them.

```mcfunction
data merge entity @n {Invlunerable:1b}
data modify entity @n Invlunerable set value 1b
```

Here is the conversion between SNBT and paths:

| SNBT Example                                      | Expression                            | Description                                                  |
| ------------------------------------------------- | ------------------------------------- | ------------------------------------------------------------ |
| `{foo:"example"}`                                 | `foo`                                 | Specifies the root element's child named `foo`               |
| `{foo:{bar:"example"}}`                           | `foo.bar`                             | Specifies `foo`’s child named `bar`                          |
| `{list:["element1","element2"]}`                  | `list[0]`                             | Specifies the first element of `list` (must be a list/array) |
| `{list:[{child:"value"}]}`                        | `list[0].child`                       | Specifies the child `child` of the first element             |
| `{list:["a","b","c"]}`                            | `list[-1]`                            | Specifies the last element of `list`                         |
| `{foo:{"bar with [[weird]] characters":"value"}}` | `foo."bar with [[weird]] characters"` | Specifies a child with special characters in its name        |
| `{list:["a","b","c"]}`                            | `list[]`                              | Gets every element of `list`                                 |
| `{list:[{flag:1b},{flag:0b},{flag:1b}]}`          | `list[{flag:1b}]`                     | Gets elements where `flag` equals `1b`                       |
| `{}`                                              | `{}`                                  | Specifies the root element                                   |
| `{name:"mcc",cool:1b}`                            | `{name:"mcc",cool:1b}`                | Matches the root if it equals this compound                  |
| `{user:{name:"sky"}}`                             | `user{name:"sky"}`                    | Specifies `user` if `user.name` is `"sky"`                   |

