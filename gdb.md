
## breaks
add:
break <[filename:]linenum|[filename:]function|*0x<address>>
remove:
clear <[filename:]linenum|[filename:]function>
info:
info break


## navigate

next|n [N]: next N source line, do not step into
step|s [N]: next N source line, step into calls

nexti / ni | stepi / si: next instruction

## dump

x
x <addr>
x/<n><f><u> <addr>

n {int}, repeat count
    how much memory of <u> units to display. if < 0, examine backwards
f {char}, format
    possibles displays are thoses used by printf (‘x’, ‘d’, ‘u’, ‘o’, ‘t’, ‘a’, ‘c’, ‘f’, ‘s’) and 'i' for machine instruction. default to 'x'
u {char}, unit
    any of b (Bytes), h (Halfwords, two bytes), w (Words, four bytes [default]), g (Giant words, eight bytes).

addr, starting display address
    0x000, $reg, *var

## stack

frame [<addr>]

addr, the address or number of stack frame
    number is from 0 (main) to the current func ?
    address if where ebp starts ?

## get @

info address <symbol>

symbol, function or variable name

## print

print|p[/<f>] [&|*]<addr|var|function|$reg>

f   format (x, a ...)

## Vars

`info variables` to list "All global and static variable names".
`info locals` to list "Local variables of current stack frame" (names and values), including static variables in that function.
`info args` to list "Arguments of the current stack frame"

0x50    0x22    0xb3    0xcf    0xfe    0x7f
7ffecfb32250