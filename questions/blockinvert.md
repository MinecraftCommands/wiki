# Do something if a command block *wasn't* successful

> [!NOTE]
> in 1.13+ we can use `execute unless ...` instead

(This is for command blocks only. For functions, [see here](/wiki/questions/functionconditions))

> [!NOTE]
> You could use a redstone torch and a comparator, but it is **not** recomended as it causes more lag than the other methods described below.

When a command block runs, its `SuccessCount` tag is updated. This is the value used by comparators to decide how strong a signal to output from that command block. If the command block does not succeed, its `SuccessCount` will be `0`.

We can use another command block to check whether the first block has `SuccessCount:0` after running, so that this second block will be successful when the first block fails, then run our conditionals off of this second block. The commands would look like:

    # pre 1.13
    testfor @a[r=5]
    testforblock ~ ~ ~1 command_block * {SuccessCount:0}
    say Nobody is within 5 blocks!
    
    # 1.13+
    <any command>
    execute if block ~ ~ ~1 command_block{SuccessCount:0}
    say Command didn't succed

[Example image](http://i.imgur.com/Syq4crm.png).

The first command is whatever you want to check wasn't successful.  
In the second command you must adjust `~ ~ ~1 command_block` to match the location and type of the first block. For example, it may be `~-1 ~ ~ chain_command_block`.  

The third command is conditional, and is whatever you want to happen when the first command fails.
