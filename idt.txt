## Interrupt Descriptor Table

array describing the interrupt vector table and stored in the IDTR(egister) on the processor
used to signal interrupts from hardware (or software) to the kernel

The IDT can contain up to 256 handlers :
- the firsts 32 are exceptions from the CPU
- generally, from 0x70 to 0x77 then 0x77 to 0x7F comes Interrupts Request (IRQ) or hardware interrupts
  two types exists today :
	- IRQ Lines, or Pin-based IRQs : a physical chip named IRQ controller (Intel 8259) queue all the hardware interrupts and
	  send them one by one to prevent race conditions. The IRQ handling is made by in/out instruction to communicate with
	  the IRQ controller (get IRQ / ack IRQ)
	- Message Based Interrupts : IRQs have memory maped addresses where are written informations about the interrupt device, interrupt that occured itself, and vectoring informations.
	  Then an IRQ is generated on the bus (PCI bus)
- everywhere else the kernel wants are software interrupts, these are used to get the kernel attention from a running process on the CPU (INT instruction)
  and are called "system calls"

each interrupt has a handler (function) assigned that is called when it occures, the structure of an IDT entry is as follow (32 bits):

0  1  2  3  4  5  6  7  8  9  10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 61 62 63
| handler address (offset) low 61 bits 0:15    | code segment selector location int he GDT     | unused all 0          | gate type |SS| DPL |P | handler address high 16 bit 16:31             |

offset 32 bits: address of the interrupt handler
selector 16 bits: index of the code segment to use in the GDT
unused 8 bit: all 0
type 4 bits: Gate type
	- 0101b | 0x5 | 5 => 80386 32 bit task gate
	- 0110b | 0x6 | 6 => 80286 16 bit interrupt gate
	- 0111b | 0x7 | 7 => 80286 16 bit trap gate
	- 1110b | 0xE | 14 => 80386 32 bit interrupt gate
	- 1111b | 0xF | 15 => 80386 32 bit trap gate
SS 1 bit: storage segment, set to 0 for interrupt and trap (only used for task gate ?)
DPL 2 bits: ring level (0-3), used for gate call protection, specifies the privilege level the calling descriptor should have at minimum.
P 1 bit: present flag, 0 for unused interrupts

Interrupt gates clear the Interrupt Flag #IF in CFLAGS before the handler is called.
This means it's useless to "cli" in the handler

### LIDT
0:15 size (limit) of the IDT - 1 | 16:47 base (address) of the IDT in memory