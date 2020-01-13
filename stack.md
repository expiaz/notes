The stack grows downwards
from the lower address to the higher address

Because the stack grows downwards, to access the 'var' we'll need
[ebp - 4] or [esp]. Because the processor will read data from 0x0ffc to 0x1000

x1000  _____  ebp 
x0ffc | var | esp (4 bytes interger)
      |     |
x0    |_____|

EBP represents to (Extended) Base Pointer, so the base of the stack
ESP stands for (Extended) Stack Pointer, so the top of the stack

 @    top
0x0  _____  esp ^
    |_____|     |
    |_____|     |
0xN |_____| ebp |
      bot

But generally we reprensent it upside down because for implementation reasons, the stack grows downwards (reaching address 0 at the top)

 @   bot
0xN  _____  ebp  |
    |_____|      |
    |_____|      |
0x0 |_____| esp  v
      top