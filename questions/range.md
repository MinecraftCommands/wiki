# Ranges

Multiple selectors (e.g. `scores` or `distance`) allow for you to specify a **range** of values to test for, instead of just a single value. They are denoted by two dots which seperate the min and max value (including): `min..max`  
Either one can be left out to signify an open-ended range. Leaving writing a single number signifies an exact check for this value alone.

`1` means "exactly 1"  
`1..` means "1 or more"  
`..1` means "1 or less"  
`1..9` means "1 to 9"  

Some selectors (like `distance`) also allow for decimal numbers. `0.5..0.9` works fine in those instances.

| ⚠️ Important |
|--------------|
|Ranges are not useable when checking for NBT!|
