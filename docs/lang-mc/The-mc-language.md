# Contexts
- Folder context refers to where functions and directories can be defined.
- Function context refers to syntax inside of functions where commands can be written.
- Macro context refers to the folder context inside of `.mcm` files
```
#!foo.mc
# Folder context
dir {
  # Folder context
  function bar {
    # Function context
    say Hello
  }
}
# Folder context
function bas {
  # Function context
  say World!
}
```
# Functions
You can create functions using the `function` keyword:
```
function foo {
  say I am called via `function mc_file_name:foo`
}
```
Functions can only be defined within [Folder context](https://mcbuild.dev/docs/lang-mc/The-mc-language#contexts)

# Directories
You can create sub directories using the `dir` keyword:
```
dir foo {
  function bar {
    say I am called via `function mc_file_name:foo/bar`
  }
  dir bar {
    function bas {
      say I am called via `function mc_file_name:foo/bar/bas`
    }
  }
}
```
Directories can only be defined within [Folder context](https://mcbuild.dev/docs/lang-mc/The-mc-language#contexts)

# Function Blocks
Function blocks are used to define groups of commands that will be wrapped up into a function on compile-time.

Using the `block` keyword will define a block and inline the generated function call:
```
block {
  scoreboard players set @s value 1
  tellraw @s {"score":{"objective":"value","name":"@s"}}
}
# When this compiles it will inline something similar to this:
# function mc_file_name:__generated__/block/0

# The block keyword can be omitted, but recommended for readability in most cases.
# This will compile the same as above
{
  scoreboard players set @s value 1
  tellraw @s {"score":{"objective":"value","name":"@s"}}
}
```
Function blocks can also be defined at the end of execute commands:
```
execute if score @s value matches 1 run block {
  tellraw @s "Value is 1"
}
# When this compiles it will inline something similar to this:
# execute if score @s value matches 1 run function mc_file_name:__generated__/execute/0

# Again the block keyword can be omitted:
execute if score @s value matches 1 run {
  tellraw @s "Value was 1"
  scoreboard players set @s value 0
}
```
Keep in mind that if you do not omit the `block` keyword after an `execute` command it will always define a function, even if the block only contains a single command. If omitted and only 1 command is inside the block instead of making a function it will just append that command to the end of the `execute` command. For example:
```
execute if score @s value matches 1 run {
  tellraw @s "Value is 1"
}
```
Would compile to `execute if score @s value matches 1 run tellraw @s "Value was 1"`

Function blocks can also be named using the `name` keyword:
```
block {
  name foo
  say hi
}
```
This allows for manual naming to remove the `__generated__` folders from your datapack if you so choose.

Function blocks can only be defined and used within [Function context](https://mcbuild.dev/docs/lang-mc/The-mc-language#contexts)

# Clocks
Clocks are an easy way of running a function every x amount of time.
For instance you could create a clock that runs every 5 seconds like this:
```
clock 5s {
  say I run every 5 seconds!
}
```
The time argument takes a `schedule` command time input (`5s`, `10t`, `1d`)

Clocks can only be defined and used within [Folder context](https://mcbuild.dev/docs/lang-mc/The-mc-language#contexts)

# While, Async While, and Until
### While
The `while` loop can be used to run a set of commands while a condition (defined as an `execute` subcommand chain) is true. It will halt the execution of the function until the loop exits:
```
scoreboard players set #i 0
while(if score #i value matches ..10) {
  tellraw @a [{"text":"loop "},{"score":{"name":"#i","objective":"value"}}]
  scoreboard players add #i value 1
}
say I will run once the while loop has ended!
```
This will loop until the value of `#i` is greater than `10`, and then it will exit the loop and continue running the function

### Async While
The async while loop will run a set of commands every x amount of time until it's condition is true without halting the execution of the function.
```
scoreboard players set #i value 0
async while(if score #i value matches ..10,5t) {
  scoreboard players add #i value 1
  tellraw @a [{"text":"loop "},{"score":{"name":"#i","objective":"value"}}]
}
say I will run immediately after the first while loop
```
The `async while` loop can also have a `finally` function block defined that will execute it's contents once the loop exits:
```
scoreboard players set #i value 0
async while(if score #i value matches ..10,5t) {
  tellraw @a [{"text":"loop "},{"score":{"name":"#i","objective":"value"}}]
  scoreboard players add #i value 1
} finally {
  say I will run once the loop exits!
}
say I will run immediately after the first while loop
```

### Until
The `until` loop will run a set of commands every x amount of time until the condition is met. It will not halt function execution:
```
scoreboard players set #i value 0
until(if score #i value matches 10,1s) {
  tellraw @a [{"text":"loop "},{"score":{"name":"#i","objective":"value"}}]   
  scoreboard players add #i value 1
}
```
`blocking until` and the `finally` block are planned but not implemented as of yet.

while, async while, and until can only be defined and used within [Function context](https://mcbuild.dev/docs/lang-mc/The-mc-language#contexts)

# Multiline Commands
*WARNING* This feature is *experimental*. It may not always work the way you expect

You can use `()` curved brackets to surround a command to allow newlines in that command:
```
(
  execute
  if block ~ ~ ~ stone
  if block ~ ~1 ~ stone
  run say hi
)
# This would compile to:
execute if block ~ ~ ~ stone if block ~ ~1 ~ stone run say hi
```

Multiline commands can only be defined and used within [Function context](https://mcbuild.dev/docs/lang-mc/The-mc-language#contexts)

# Inline & Multiline JS Blocks
### Inline JS block
Inline JS blocks are JS code wrapped in `<%` and `%>`. Whatever the JS inside of those brackets returns will be what is inserted into the command/line:
```
# For instance if the value of config.text is "hello" than this code:
say <%config.text%>
# Will compile to this:
say hello
```
Inline JS blocks cannot span multiple lines.

### Multiline JS block
Multiline JS blocks are JS code wrapped in `<%%` and `%%>`.
```
<%%
  for (let i=0; i<10; i++) {
    emit(`say ${i}`)
  }
%%>
```
The opening and closing brackets must be the only thing on their line. Only whitespace is allowed before them

Inline and Multiline JS blocks can only be defined and used within [Function context](https://mcbuild.dev/docs/lang-mc/The-mc-language#contexts)

# Compile time logic
### !IF
The compile time if statement can be used to omit code and/or functions if a certain condition is not met. For example if you didn't want a dev function to export unless a flag was true in the config you could do something like this:
```
!IF(config.dev) {
  function dev_tool {
    say I'm a dev function. Yay!
  }
}
```
This would only generate that function IF config.dev is set to `true`.
### Shorthand !IF
Same as !IF, but less to type:
```
!config.dev{
  say hi
}
```
### LOOP
Compile time LOOP allows you to generate code based off of a JS iterable value. The second argument is the loop variable that will be passed to the context inside the loop.
```
# This will generate 10 functions, loop_0-9 that say their name when ran
LOOP(10,i){
  function loop_<%i%> {
    say loop_<%i%>
  }
}
```

Compile time logic (!IF shorthand compile time if, and LOOP) can be defined and used within [Function and Folder context](https://mcbuild.dev/docs/lang-mc/The-mc-language#contexts)

# Execute if else blocks
Execute if else blocks allow you to write run-time if else logic with ease:
```
execute(if score #i value matches 1){
  say The value is 1
}else execute(if score #i value matches 2){
  say The value is 2
}else{
  say I don't know what value it is, but it ain't 1 or 2 ',:I
}
```

Execute if else blocks can be defined and used within [Function context](https://mcbuild.dev/docs/lang-mc/The-mc-language#contexts)


# Macros




