The stack grows downwards
from the lower address to the higher address

Because the stack grows downwards, to access the 'var' we'll need
[ebp - 4] or [esp]. Because the processor will read data from 0x0ffc to 0x1000

x1000  ____ ebp 
x0ffc | var esp (4 bytes interger)
	  |
x0	  |____

