---
title: Lua Quickstart
---

> ~ 5 minute read

This is an absolute beginners guide to programming. This will only cover the absolute basics you need to understand. If you want to learn more about Lua check out the [Lua Documentation](https://www.lua.org/pil/1.html). Here is a [Table of Contents](https://www.lua.org/pil/contents.html#1).

## How a computer reads code

Computers read code line by line from the top to the bottom of the file. Each line is just a command telling the computer what to do.

We will now cover the most important concepts you need to know.

## Variables

This is how you create a variable:

```lua
myVar = 3
```

On the left side you put the variable name, in this case I just called it "myVar". Then put a ``=`` symbol and on the right side put the value you want to assign. This basically just stores the value `3` inside your variable. Whenever you use this same variable again, it will use this value to do things.

Note: You can make comments in your code by putting two minus symbols, anything right of that gets ignored, this is good for descriptions or putting notes.

```lua
num1 = 3 -- Creates a variable and assigns it the value 3
num2 = 7 -- Creates another variable and assigns it the value 7
sum = num1 + num2 -- Store the sum inside another variable
print(sum) -- Prints "10"
```

## Functions

Functions are useful for things you want to use multiple times in your code. Instead of re-writing the code every time, instead you can just call the function and any code inside of it will be executed. (To "call" a function basically just means to execute it)

```lua
function test()
    -- anything you put inside here will be executed when the function is called
end
```

The `function` keyword lets us create a function. After it, you put the name of the function, in this case we named it "test". Then you put `()` after it. You close this block by putting the `end` keyword below it.

You can put variable names inside the `()` to pass a variable to the function. These variables are called function parameters. If you have multiple you can just separate them with a comma `,` character.

In this function, we just print a message depending on what name you give to the function. (To tell the computer that some text is just supposed to be a string instead of a command surround it with `"`. To concatenate strings together use `..`)

```lua
function greeting(name)
    print("Hello " .. name)
end
```

Now somewhere else in your code you can call the function like follows:

```lua
greeting("Barry") -- > Hello Barry
-- you can also use a variable instead:
person = "Cynthia"
greeting(person)  -- > Hello Cynthia
```

Functions can also return a value, here is an example:

```lua
function sum(a,b)
    local c = a+b -- Use the local keyword to make a variable that is only used inside of this block
    return c -- Use the return keyword to give back a value to the caller
end

-- Now somewhere else in your code:
result = sum(3,5)
print(result) -- Prints "8"
```

## Conditions

In order to do something under some condition we use the `if` keyword. You can think of it as a question, "if that happens do this, otherwise do something else".

```lua
if condition then
    -- do something if condition is true
else
    -- if not, do something else
end
```

Similar to functions, we have whats called a code block here. Two of them actually, one between the ``then`` and the ``else``, which gets executed when the condition is met, and another one between the ``else`` and the ``end`` which gets executed if the condition is not met.

Most of the time we want to compare variables in our conditions. 

```lua
num = 10
if num > 5 then
    print("This number is greater than 5")
else
    print("This number is smaller or equal to 5")
end
```

To compare variables we can use `==` to test if something is equal, `<` to test if something is smaller or `>` to test if something is greater. Note that we use double equals `==` here, because a single equals `=` symbol is used to assign values to variables. There is also `<=` for "smaller or equal to", and `>=` for "greater or equal to".

```lua
name = "Dawn"

if name == "Lucas" then
    print("Hey Lucas!")
elseif name == "Dawn" then
    print("Hello Dawn!")
else
    print("Who are you??")
end
```

Note: Its good to indent your code properly to easily tell where blocks begin and where they end.

```lua
-- good:

function test(thing)
    if thing then
        print("yes")
    else
        print("no")
    end
end

-- bad:

function test(thing)
if thing then
print("yes")
else
print("no")
end
end
```

## Figura Specific

Now that we have that out of the way, you can take a look at [what you need to know for Figura](/figura-scripts).