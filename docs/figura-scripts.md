---
title: Figura Lua Scripts
---

> ~ 3 minute read

Now that you [know how to program](/lua-quickstart), you should be able to use [the wiki](https://github.com/Blancworks/Figura/wiki) to get what you need. Some things are currently not on the wiki since its a little outdated, so we will cover these things below.

Your script is executed when your avatar loads. For you this happens when you equip the avatar, for other players when you enter their render distance.

Most importantly, if your avatar loads your code will only be executed once, however most of the time you want to continuously check your player's state and act accordingly to it.

If you name your functions exactly `tick` or `render`, Figura will call them regularly. There is also `world_render` which is almost the same as `render` except that it will also run if your model is not in the player's view.

```lua
function tick()
    -- Figura will call this function every tick (20 times a second)
end

function render(delta)
    -- Figura will call this function every frame when your model is visible
    -- (Therefore depends on your fps)
end
```

You will do most stuff inside the `tick` function. Note that you can have multiple tick or render functions (Unlike regular functions which names have to be unique).

Next, we have the `model` tree. This will give you access to your BlockBench model. It is structured the exact same way the outliner in BlockBench shows your cubes and groups. Figura provides a variable called `model` which contains all your modelparts.

![outliner-1](/assets/outliner-1.png)

```lua
model.Head.Hat.setEnabled(false) -- hides the hat
```

### Action Wheel and Keybinds

The action wheel allows us to bring up a wheel by pressing `B` and we can attach functions to the wheel slots.

```lua
action_wheel.SLOT_1.setItem("minecraft:grass_block")
action_wheel.SLOT_1.setTitle("Click me :D")
action_wheel.SLOT_1.setFunction(function () -- anonymous function, does not have a name
    print("Hooray!")
end)
```

Now if you select the first slot, it will print "Hooray!" in chat.

If you want something to happen on a key press (List of keys can be found [here](https://discord.com/channels/805969743466332191/808155531389698079/854763353855885364)):

```lua
myKey = keybind.newKey("Send Hooray Button", "K")

function world_render()
    if myKey.wasPressed() then
        print("Hooray!")
    end
end
```

Important! Other players don't know that you clicked the slot! To send the information about the click to other players we use Figuras pings system.

### Pings

Pings are used to send information that is only available to the host (you) to all other players. To make a ping function make a function inside the `ping` table like so:

```lua
function ping.setHat(x)
    model.Head.Hat.setEnabled(x)
end
```

Whenever you call ``ping.setHat()`` this function will be executed on all instances of your script (so on all players including yourself). Calling a ping function only does something on the host instance of the script, other players seeing your model and running the script just ignore this line of code.

Since action wheel and keybinds are only available to you, the host script instance, you will want to ping everyone else to also execute the action bound to it:

```lua
action_wheel.SLOT_2.setTitle("Hide Hat")
action_wheel.SLOT_2.setFunction(function ()
    ping.setHat(false)
end)
```
