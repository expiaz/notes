## GDT

Global Descriptor Table

contains segments descriptions and stored into GDTR(egister) into processor

used to declare segments of memory that each have a different purpose (e.g. Code, Data ...)
with different rights (which ring, write exec read ...)

This methods was used to define kernel and process memory space physically, butit has his limitations
- only a fixed number of segments can be declared (up to the maximum amount of memory)

the advantage of segmentation is that you can set the ring level, and you can't with paging
so the goal is to set up segments for the kernel in ring 0 and segments for the userland in ring 3
to use later with paging

## segmentation registers

they hold the address of the start of the corresponding segment in memory :
- cs (code segments) => code flag (x)
- ds (data segments) => data flag (rw)
- es (extra segment) => data flag
- fs, gs
- ss (stack segment) => data flag

To define our segments we'll use one entry per segment wanted, entries looks like these :
THE FIRST ENTRY MUST BE NULL (zeroed)

gdt entry : 64 bits
 0  1  2  3  4  5  6  7  8  9  10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 61 62 63
| size (limit) low 16 bits (0:15)               | start address (base) low 16 bits (0:15)       | base mid 8 (16:23)    |P | DPL |DT|X |DC|RW|AC|G |D |0 |A |limit 16:19| base high 8 (24:31)   |

limit 20 bits: size of the segment
base 32 bits: address of the segment
access 8 bits (40:47):
- P 1 bit (present): is segment present (1 = y, 0 = n)
- DPL 2 bits (descriptor privilege level): ring 0-3
- DT 1 bit (Descriptor Type): always 1 ?
- Type 4 bits:
	- E: Executable bit. If 1 code in this segment can be executed, ie. a code selector. If 0 it is a data selector.
	- DC: Direction bit/Conforming bit.
		Direction bit for data selectors: Tells the direction. 0 the segment grows up. 1 the segment grows down, ie. the offset has to be greater than the limit.
		Conforming bit for code selectors:
			- If 1 code in this segment can be executed from an equal or lower privilege level. For example, code in ring 3 can far-jump to conforming code in a ring 2 segment. The privl-bits represent the highest privilege level that is allowed to execute the segment. For example, code in ring 0 cannot far-jump to a conforming code segment with privl==0x2, while code in ring 2 and 3 can. Note that the privilege level remains the same, ie. a far-jump form ring 3 to a privl==2-segment remains in ring 3 after the jump.
			- If 0 code in this segment can only be executed from the ring set in privl.
	- RW: Readable bit/Writable bit.
		Readable bit for code selectors: Whether read access for this segment is allowed. Write access is never allowed for code segments.
		Writable bit for data selectors: Whether write access for this segment is allowed. Read access is always allowed for data segments.
	- AC: Accessed bit. Just set to 0. The CPU sets this to 1 when the segment is accessed.
granularity 4 bits (48:51):
- G 1 bit (granularity): 0 = 1 byte (8 bits), 1 = 1 kbyte (8000 or 8096 ?)
- D 1 bit: operande size (0 = 16bit, 1 = 32bit)
- 0 1 bit: should always be zero
- A 1 bit: available for system use (should always be zero)

### LGDT

ldgt takes the address of a struct describing the GDT in memory

the struct is : 0:15 size (limit) of the GDT - 1 | 16:47 base (address) of the GDT