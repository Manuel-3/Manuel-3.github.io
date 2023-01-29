---
title: Custom Armor
article: true
---

**THIS GUIDE IS FOR FIGURA ALPHA! WHILE THE PRINCIPLES STILL APPLY, THE EXAMPLE CODE WILL NOT WORK FOR BETA OR NEWER VERSIONS!**

There are several ways to make custom armor. Let's take a look at all of them:

- Resizing/Repositioning vanilla armor
- Custom armor using custom texture
- Custom armor using vanilla textures
- Unique shapes for different materials

# Resizing/Repositioning vanilla armor

If your model is humanoid and has roughly the same proportions of the vanilla player, it might be enough to just slightly move the vanilla armor around. To do this, simply use `setPos` and `setScale` like so:

```lua
armor_model.HELMET.setPos({0,-3,0})
armor_model.HELMET.setScale({0.9,0.9,0.9})
-- Instead of HELMET you can also use
-- CHESTPLATE
-- LEGGINGS
-- BOOTS
```

# Custom armor using custom texture

First, don't forget to hide the vanilla armor.

```lua
for _, v in pairs(armor_model) do
    v.setEnabled(false)
end
```

If you want custom armor, simply make cubes for it in BlockBench. Make sure that they are inside your keyword groups so they also move along with the body parts. A simple way to make armor is to duplicate a cube and using the inflate value to make it a little bigger. Then move it's UV to a new spot where you can put the armor texture.

Here are the inflate values you should be using:

Helmet | 1
Chestplate | 1.01
Arms | 1
Belt | 0.51
Legs | 0.5
Boots | 1

Here is an example on how to make a helmet:

![Duplicate the Head](./assets/blockbench-1.png)
![Change UV](./assets/blockbench-2.png)

It might look a little big in BlockBench, but in game it will actually look fine:

![What it looks like in game](./assets/minecraft-4.png)

After you made your complete set of custom armor, you can then add multiple materials for it to switch to. I have added a chestplate and 4 materials for demonstration. Note that you can use the "Mirror UV" button for the left arm or leg to flip the texture the correct way.

![More materials](./assets/blockbench-3.png)

Now we will make a script that enables or disables our parts whenever we wear armor, and also checks what kind of armor we are wearing to decide what material to show.

Here is a really handy utility function for selecting UV offsets according to a material. Just put the correct UV offsets into the `materials` table. Since our UV is originally at diamond, we will put `{0,0}` for it. Then just fill in the other ones. This even works with modded materials if you wanted to.

```lua
-- These values are later used for setUV
materials = {
    ["diamond"]={0,0},
    ["golden"]={56/192,0},
    ["netherite"]={56/192,32/64},
    ["leather"]={0,32/64},
}

-- This function checks the name of an item to return the
-- corresponding material from the table above.
function material(item)
    local name = item:sub(item:find(":")+1)
    local n = name:find("_")
    local mat = name:sub(0,n~=nil and n-1 or name:len())
    local uv = materials[mat]
    return (uv~=nil and uv or {0,0})
end
```

For example, if we now call `material("minecraft:netherite_helmet")` or `material("minecraft:netherite_chestplate")` it will return `{56/192,32/64}` both times.

You can obviously add more materials if you'd like.

Now let's actually make the armor work, shall we?

```lua
function tick()
    -- First get all the equipped armor pieces
    local helmet = player.getEquipmentItem(6)
    local chestplate = player.getEquipmentItem(5)
    local leggings = player.getEquipmentItem(4)
    local boots = player.getEquipmentItem(3)

    -- Move UV according to the material
    model.Head.Helmet.setUV(material(helmet.getType()))
    model.Body.Chestplate.setUV(material(chestplate.getType()))
    model.RightArm.Chestplate.setUV(material(chestplate.getType()))
    model.LeftArm.Chestplate.setUV(material(chestplate.getType()))

    -- Enable armor when wearing something, disable when not
    model.Head.Helmet.setEnabled(helmet.getType()~="minecraft:air")
    model.Body.Chestplate.setEnabled(chestplate.getType()~="minecraft:air")
    model.RightArm.Chestplate.setEnabled(chestplate.getType()~="minecraft:air")
    model.LeftArm.Chestplate.setEnabled(chestplate.getType()~="minecraft:air")

    -- You can also add enchantment glint
    model.Head.Helmet.setShader(helmet.hasGlint() and "Glint" or "None")
    model.Body.Chestplate.setShader(chestplate.hasGlint() and "Glint" or "None")
    model.RightArm.Chestplate.setShader(chestplate.hasGlint() and "Glint" or "None")
    model.LeftArm.Chestplate.setShader(chestplate.hasGlint() and "Glint" or "None")
end
```

Now this almost works, the only thing missing is the leather color.

![Leather not colored](./assets/minecraft-5.png)

We can use the following function to get the color of an armor piece:

```lua
function getColor(stack)
    if not stack.getType():find("leather") then return {1,1,1} end
    local tag = stack.getTag()
    if tag ~= nil and tag.display ~= nil then
        return vectors.intToRGB(tag.display.color)
    end
    return {160/255, 101/255, 64/255} -- default leather color
end
```

With that, we can add coloring to the bottom of our tick function:

```lua
model.Head.Helmet.setColor(getColor(helmet))
model.Body.Chestplate.setColor(getColor(chestplate))
model.RightArm.Chestplate.setColor(getColor(chestplate))
model.LeftArm.Chestplate.setColor(getColor(chestplate))
```

And there we go!

![Leather colors](./assets/minecraft-6.png)

# Custom armor using vanilla textures

Alternatively, you can use the vanilla armor textures. This makes it so your armor works with resource packs and you also don't have to use space on your texture.

For this it is useful to import one of the texture files into your BlockBench project, in order to figure out the proper UV that is needed. IMPORTANT: The model is **not** going to use that importet texture! Figura only supports one texture at a time (for now), and it is going to read whatever texture you saved to the `texture.png` file, **not the textures inside your BB project!** It is just for reference, you can even delete it afterwards, it won't matter.

As you can see, the UVs are a little stretched (double it's height to be exact). This is because the armor texture file is 32 pixels high, but the models texture is 64 pixels high. To stretch individual faces you must be using per-face-UV mode.

![More materials](./assets/blockbench-4.png)

In our script, instead of UV offsets we will now be using vanilla texture file names:

```lua
materials = {
    ["diamond"]="minecraft:textures/models/armor/diamond_layer_1.png",
    ["golden"]="minecraft:textures/models/armor/gold_layer_1.png",
    ["netherite"]="minecraft:textures/models/armor/netherite_layer_1.png",
    ["leather"]="minecraft:textures/models/armor/leather_layer_1.png",
}
```

Then in our tick function, instead of using `setUV`, we will be using `setTexture`:

```lua
-- Select texture according to the material
model.Head.Helmet.setTexture("Resource", material(helmet.getType()))
model.Body.Chestplate.setTexture("Resource", material(chestplate.getType()))
model.RightArm.Chestplate.setTexture("Resource", material(chestplate.getType()))
model.LeftArm.Chestplate.setTexture("Resource", material(chestplate.getType()))
```

Now our armor works even with a resource pack! (I am using Faithful 32x here)

![Resource pack armor](./assets/minecraft-7.png)

# Unique shapes for different materials

Lastly, what if you wanted to make different shapes for your materials. Let's make the golden helmet look like a crown. I made a really quick crown model (these cubes sticking out just use the top face texture of the helmet):

![Crown model](./assets/blockbench-5.png)

Remember that the texture in BB is not actually going to be used by our model! Instead, add a line to our `setTexture`s (or `setUV`s, if you are using custom texture):

```lua
model.Head.CrownParts.setTexture("Resource", material(helmet.getType()))
```

Then make sure that it will only be visible if the player is wearing gold:

```lua
model.Head.CrownParts.setEnabled(helmet.getType() == "minecraft:golden_helmet")
```

![Crown in game](./assets/minecraft-8.png)
