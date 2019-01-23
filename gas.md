? = a,b,c,d (eax, ebx, ecx, edx)
=? (name) => put ? register in variable 'name' after the operation
? (name) => load ? register with 'name' before the operation

__asm__("assembly" : return : parameters)
__asm__("mov %%from, %%to" : "=to" (var) : "from" (var))
__asm__("mov %%a, %%b" : "=b" (b) : "a" (a))