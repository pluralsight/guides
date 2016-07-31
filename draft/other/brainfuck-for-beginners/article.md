## What is Brainfuck?
Despite it's name, Brainfuck is a highly esoteric programming language. Essentially, it works just using a stack, assumedly with an infinite size. The language itself is extremely simple, consisting of only 6 characters. Due to it's extremely small size, programs written in Brainfuck are extremely small, and almost impossible to read. Below is a table of the characters used, and what they do

#### Table of Commands

Command | Function of Command
:--------:|:---------------------
`>`       |Moves the pointer one cell right
`<`|Moves the pointer one cell left
`+`| Increment the cell under the pointer by one
`-`| Decrement the cell under the pointer by one
`.`| Outputs the value of the cell under the pointer as an ASCII character
`,`| Takes one character of ASCII input, and store it in the cell under the pointer
`[`| Jumps to the the matching `]` plus one instruction, if the cell currently under the pointer is 0
`]`| Jumps to the matching `[` if the cell under pointer is non-zero

### Hello World Example
To give an idea of what a Brainfuck program looks like, check out the Hello World example below
```none
 +++++ +++               Set Cell #0 to 8
[
 >++++               Add 4 to Cell #1; this will always set Cell #1 to 4
 [                   as the cell will be cleared by the loop
     >++             Add 4*2 to Cell #2
     >+++            Add 4*3 to Cell #3
     >+++            Add 4*3 to Cell #4
     >+              Add 4 to Cell #5
     <<<<-           Decrement the loop counter in Cell #1
 ]                   Loop till Cell #1 is zero
 >+                  Add 1 to Cell #2
 >+                  Add 1 to Cell #3
 >-                  Subtract 1 from Cell #4
 >>+                 Add 1 to Cell #6
 [<]                 Move back to the first zero cell you find; this will
                     be Cell #1 which was cleared by the previous loop
 <-                  Decrement the loop Counter in Cell #0
]                       Loop till Cell #0 is zero

The result of this is:
Cell No :   0   1   2   3   4   5   6
Contents:   0   0  72 104  88  32   8
Pointer :   ^

>>.                     Cell #2 has value 72 which is 'H'
>---.                   Subtract 3 from Cell #3 to get 101 which is 'e'
+++++ ++..+++.          Likewise for 'llo' from Cell #3
>>.                     Cell #5 is 32 for the space
<-.                     Subtract 1 from Cell #4 for 87 to give a 'W'
<.                      Cell #3 was set to 'o' from the end of 'Hello'
+++.----- -.----- ---.  Cell #3 for 'rl' and 'd'
>>+.                    Add 1 to Cell #5 gives us an exclamation point
>++.                    And finally a newline from Cell #6
```
(Thanks to esolang for this one)

Now, as you can see, it's not particuarly easy to read, even with the comments along side it. But what about the barebones version
```
++++++++[>++++[>++>+++>+++>+<<<<-]>+>+>->>+[<]<-]>>.>---+++++++..+++.>>.<-.<.+++.------.--------.>>+.>++.
```

Not quite so easy to read am I right