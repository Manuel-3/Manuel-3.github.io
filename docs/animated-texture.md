---
title: Animated Texture
article: true
---

Animating a texture in Figura works differently than in resource packs. Instead of a `.mcmeta` file, we use `ModelPart:setUV(u,v)` to change the texture coordinates a cube is using.

In this tutorial we are going to make a blinking animation. To make it a little easier, I have gone ahead and separated the face from the rest of the head. This is technically optional, but makes things easier because I only want the face to change, and not the entire head, since in figura we can only change UVs of the whole cube at once. (Duplicate the head cube, set the front face to transparent on the head, and all other sides except for the face to transparent on the duplicate. Make sure to be in Per-Face-UV mode for this)

![Face and Head separated](./assets/model-1.gif)

For the first part let's just use this dummy animation. To make it, I just expanded the default texture to make space at the bottom and drew a few frames. Move the UV rectangle of the face to the first frame.

![Texture UV layout](./assets/texture-1.png)

To animate it, all we have to do is make a script that cycles through these. To do this, we can use `world.getTime()` which gives us the time in game ticks, and `ModelPart:setUV(u,v)`.

The `setUV` function shifts the UV box by a given amount. It will offset it relative to the default position we have given it in BlockBench, in this case the lower left corner. Since `setUV` takes a fraction as input (1 means whole texture size, 1/2 means half of the texture size, etc..) we can just shift it 1/8 every step to make it go through all 8 frames. Since the time increases by 1 every tick, we tie the UV to `time/8` which will make it shift 1/8 each tick.

```lua
function events.tick()
    local time = world.getTime()
    models.model.Head.Face:setUV(time/8,0)
end
```
We don't even have to clamp the time value in between 0 and 7 because `setUV` just cycles around the texture if the value overflows.

![Animation in game](./assets/minecraft-1.gif)

If we want to make the animation slower, we can divide the time by some value to essentially make time go slower for our animation. Note that this will produce floating point values like 9/2=4.5, so it wouldn't shift the full 1/8 step and display some weird selection inbetween our frames. To fix this we can floor it to the next integer.

```lua
function events.tick()
    local time = world.getTime()
    models.model.Head.Face:setUV(math.floor(time/2)/8,0)
end
```

You can see that normally we displayed every frame for one tick, but if we divide the time by 2 it would display every frame for two ticks making it slower. If we write down each time value and the calculated frame we would display, it would look like this:

```
floor(time/2) = frame
floor(0/2) = 0
floor(1/2) = 0
floor(2/2) = 1
floor(3/2) = 1
floor(4/2) = 2
floor(5/2) = 2
...
```

Now let's suppose you wanted to use less frames. To prevent the UV from scrolling through the entire texture, and instead only use 4 frames for example, we can use the modulo operator, which returns the remainder you get if you would do an integer division, for example `6 % 4 = 2` because `6 / 4 = 1 with remainder of 2`.

![Texture UV layout using less frames](./assets/texture-2.png)

```lua
function events.tick()
    local time = world.getTime()
    models.model.Head.Face:setUV((time % 4)/8,0)
end
```

If you try logging `time % 4` in chat you will see that it only ranges from 0 to 3 and then starts over again even if the time is actually already much higher than 4.

![Value output](./assets/chat-1.png)

What if you had a really messy texture and no easy way to lay out your frames all next to each other though? To demonstrate, I have resized the texture back to the normal 64x64 size and I placed my frames on unused spaces. I made a simple blinking animation.

![Texture UV layout but on random free spaces](./assets/texture-3.png)

This time we are going to use pixels measurements instead. To use a pixel value, divide your value by the texture size. This is similar to before, where we wanted to step in 1/8ths, but this time if the texture size is 64 then we want to step in 1/64ths to essentially step in pixel units. For example, to shift my UV rectangle to the top right corner from where it started, it would need to be offset by 8 pixels up and 48 pixels to the right. Since the UV coordinates go "right" and "down", moving it 8 pixels up would mean we want to move it "-8 pixels down".

```lua
local width = 64
local height = 64
models.model.Head.Face:setUV(48/width,-8/height)
```

To manage the UV offsets we can create a table containing all the values, and then just cycle through them similar to what we did earlier. We wrap the values using vec() to turn them into a vector, so that we are able to pass both values at once.

```lua
local frames = {
    vec(0,0),         -- open eyes
    vec(24/64,-8/64), -- half closed eyes
    vec(-8/64,-8/64), -- closed eyes
    vec(48/64,-8/64)  -- wide open eyes
}

function events.tick()
    local time = world.getTime()
    models.model.Head.Face:setUV(frames[(time % #frames) + 1])
end
```

But where does the `frames[(time % #frames) + 1]` come from? Well, to access one of these frames positions, we do `frames[number]` for example `frames[1]` gives us `vec(0,0)` which is the first one.
Additionally `#frames` just means the number of frames we have, in this case 4. So by doing `time % #frames` or `time % 4` we again get values cycling from 0,1,2,3 and then repeat. However, we want values from starting at 1, like 1,2,3,4. So we can just add one after the previous calculation to shift them all up, giving us the desired outcome of `(time % #frames) + 1` to access our frames.

As you can see, it is now playing the blinking animation.

![Blinking animation](./assets/minecraft-2.gif)

To make it not play continuously, we can wait a bit until we restart the animation. For this its easier to make our own timer variable. (Note that my screen recorder seemingly didn't manage to get enough fps to record all frames so the gif might look like some are skipped.)

```lua
local frames = {
    vec(0,0),         -- open eyes
    vec(24/64,-8/64), -- half closed eyes
    vec(-8/64,-8/64), -- closed eys
    vec(48/64,-8/64)  -- wide open eyes
}

local time = 1
local nextBlink = 0

function events.tick()
    if time > 4 then
        time = 1
        nextBlink = world.getTime() + math.random(15,50)
    end
    models.model.Head.Face:setUV(frames[time])
    if nextBlink < world.getTime() then
        time = time + 1
    end
end
```

![Blinking animation with waiting](./assets/minecraft-3.gif)
