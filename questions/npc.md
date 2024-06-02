# How to create / customize an NPC?
This article will explain how to create NPCs and add commands to it.

> [!NOTE]
> This is for the entity in Bedrock edition called [`NPC`](https://minecraft.wiki/w/NPC)

## Basics of an NPC

For spawning an NPC, you will need to use the `/summmon` or `/give` command:

    /summon npc
    /give @s spawn_egg 1 51

For removing the NPC just left click it while you are in creative, or use the `/kill` command.
A NPC is invulnerable if you try to kill it with other methods (unless when falling into the void), but it can be moved with water, lava, breeze's wind charges, explosions or pistons.

To edit an NPC right click it in creative, you will open a GUI that allows you to edit the display name and the skin.

The NPC will be staring at the nearest player that is not in spectator, if it’s in a 6 block radius.

## Dialogues and commands

When you interact with the NPC in creative, you will find a button called `Edit Dialog`. When you click it, you will be able to edit the dialogue. The maximum of characters you can have is 307.
You can **not** use target selectors in the dialog, for example, if the dialog is `@p`, the dialog will be `@p`, not the nearest player.

At the bottom of the edit screen, there is a button called “Advanced Settings”, when you press it you will find a screen similar to the command block one, there you will write the command.
You can use a target selector variable to target the player that is “talking” to the NPC, it is `@initiator`, for example the command `/kill @initiator` will kill the player that is talking to the NPC.

For that command there are 3 options: “On Enter” to run the command when the player right clicks it, “On Exit” to run the command when the player leaves the NPC screen and “Button Mode”. 
This option will show a button in the dialog screen when the player interacts with it. You can change the button’s text by typing it in the text box below the options.
You can add another command at the same button (or action, if you choosed it to be `On Enter`, for example) by pressing `enter` to create a new line.

The command is run `as` the NPC and `at` the NPC, see [command context](wiki/questions/commandcontext) for more information.

Below the options, you can choose to add another command, to remove a command press the trash button in the top right corner of that command.

## Add another dialog
If you want you can add another dialog when the player presses a button.

Example: the NPC asks if the player wants a challenge and there are 2 options, yes and no. If the player selects no, no action is initiated. However, if the player selects yes, a dialog will appear inquiring if they are certain about this with 2 options again: yes and no. If they select yes, a zombie will spawn.

This is done thanks to the `/dialogue` command, that opens NPC dialogues.

First NPC (The one player will interact with), with the dialog “Do you want a challenge?”

    # Button mode with the text “Yes”
    dialogue open @e[tag=npc.confirm,type=npc] @initiator
    
    # Button mode with the text “No”
    # This is empty as when pressing any button, the NPC dialogue will close.

Now we will hide a second NPC underground (so player will **not** see it)  with the tag `npc.confirm`, With the dialogue “Are you sure?”

    # Button mode with the text “Yes”
    execute at @initiator run summon zombie
    say here is your challenge

    # Button mode with the text “No”
    # This is empty as when pressing any button, the NPC dialogue will close.

And now you have a dialog that will pop up when you press a button. You can add more dialogues with more NPCs, but each one needs a different tag to identify it.
