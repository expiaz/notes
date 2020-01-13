c calling convention

call func				; goto callee
    push eip+sizeof(*eip)
    jmp func

enter                   ; create new stack frame (backup old one before)
    push ebp
    mov ebp, esp

leave                   ; delete local variables
    mov esp, ebp
    pop ebp

ret						; return to caller
    pop return
    jmp return


convention:

call func
enter
body
leave
ret

--caller
push ip+1
jmp func
--callee
push ebp
mov ebp, esp
sub esp, 0x4
--body local
mov [ebp-0x4], dword 0x5 ; mov [esp], dword 0x5
--ret value
mov eax, [ebp-0x4]
mov esp, ebp
pop ebp
pop ebx
jmp ebx
--caller

int callee(int a, int b, int c)
{
	int add;

	add = a + b;
	return add + c;
}

int caller(void)
{
	int var;
	
	return var = calle(1, 2, 3) + 5;
}

/!\ esp and ebp points to the first bit of the represented data
e.g. [esp] == 1 && [ebp] == 3

in little edian data are stored from 0 to inf.
e.g. that's why [ebp-4] == 3 and not [ebp] == 3 
bc [ebp] is data from where is ebp to the size of the data accessed

x1000  ____   ebp 
x0ffc | var | esp (4 bytes interger)
	  |     |
x0	  |____ |

1. passing parameters from caller

push call arguments, in reverse (some compilers may subtract the required space from the
stack pointer, then write each argument directly, see below. The 'enter' instruction can also do something similar)

sub esp, 12 ; 'enter' instruction could do this for us
mov [ebp-4], 3 ; or mov [esp+8], 3
mov [ebp-8], 2  ; or mov [esp+4], 2
mov [ebp-12], 1  ; or mov [esp], 1

; then we jump to the function but save our return address before
push eip + 1
jmp callee

OR

push    3
push    2
push    1
call    callee

x1000  _____  ebp
	4 | var |
	4 | 3   |
	4 | 2   |
	4 | 1   |
	@ | ret | esp
0	  |_____|

2. Creating new stack frame for callee

push ebp	; save old call frame bc callee is responsible to recreate it after
mov  ebp, esp	; initalize new call frame (0 data for the moment so bp === sp)

x1000  _____
	4 | var |
	4 | 3   |
	4 | 2   |
	4 | 1   |
	@ | ret |
	@ | ebp | ebp, esp
0	  |____ |

3. Local variables and accessible parameters

sub	esp, 4 ; make space for the local variable 'add'

mov	eax, [ebp + 4] ; parameter a
mov	ebx, [ebp + 8] ; parameter b

add	eax, ebx     ; a + b
mov	[esp], eax   ; add = a + b

mov	ebx, [ebp + 12] ; parameter c

mov eax, [esp]
add	eax, ebx     ; add + c

x1000  _____
	4 | var |
	4 | 3   |
	4 | 2   |
	4 | 1   |
	@ | ret |
	@ | ebp | ebp
	4 | add | esp
0	  |_____|

4. Returning from the function

restore old call frame (some compilers may produce a 'leave' instruction instead)

add esp, 4		; remove local variables from frame, ebp - esp = 4.
OR
mov esp, ebp	; compilers will usually produce the following instead
				; which is just as fast,
				; and, unlike the add instruction
				; also works for variable length arguments
				; and variable length arrays allocated on the stack.

x1000  _____
	4 | var |
	4 | 3   |
	4 | 2   |
	4 | 1   |
	@ | ret |
	@ | ebp | ebp, esp
	4 | add |
0	  |_____|


most calling conventions dictate ebp be callee-saved,
i.e. it's preserved after calling the callee.
it therefore still points to the start of our stack frame.
we do need to make sure callee doesn't modify (or restores) ebp, though,
so we need to make sure it uses a calling convention which does this

pop ebp ; restore the caller base stack

x1000  _____  ebp
	4 | var |
	4 | 3   |
	4 | 2   |
	4 | 1   |
	@ | ret | esp
	@ | ebp |
	4 | add |
0	  |_____|

ret ; return add + c;
OR 
pop ebx ; get the return address from the stack
jmp ebx

x1000  _____  ebp
	4 | var |
	4 | 3   |
	4 | 2   |
	4 | 1   | esp
	@ | ret |
	@ | ebp |
	4 | add |
0	  |_____|


5. use return value

add eax, 5 ; callee(1, 2, 3) + 5

6. clean parameters

add esp, 12 (4 x 3)

x1000  _____  ebp
	4 | var | esp
	4 | 3   |
	4 | 2   |
	4 | 1   |
	@ | ret |
	@ | ebp |
	4 | add |
0	  |_____|
