[https://www.student.cs.uwaterloo.ca/~cs350/W11/notes/vm.pdf]
[https://web.archive.org/web/20160326061042/http://jamesmolloy.co.uk/tutorial_html/6.-Paging.html]
[https://wiki.osdev.org/Memory_management]
[https://wiki.osdev.org/Paging]

# Virtual Adress Space

Physical addresses are provided directly by the machine.
1 physical address space per machine
the size of a physical address determines the maximum amount of addressable physical memory
  e.g. 32bits => 2^32 => 4GB max

Virtual addresses (or logical addresses) are addresses provided by the OS to processes.
one virtual address space per process
=> as the program runs, the OS and hardware (MMU) translate virtual @ to physical @
   => @ddress translation

A virtual address space starts at 0 and has the amout of memory of the machine.
This makes it easier for programms who can assert positions in the code and data from their offsets
in the binary.
Using physical addresses would need to calculate the offset from the start of the program to 
compute the addresses of functions and data in the code.

Physical layout of 2 processes:
|---------|       	|-----------|
A		  			B

0              0x100             0x259                        0xFFF
|---------------|---------|--------|-----------|----------------|
                A         A        B           B               end
Processes A & B have separated adresses spaces, one starting at 0x100 and the other at 0x259.
Their memory is contiguous, and have a different starting point at each loading in memory.
The offsets in the code of these processes need to be calculated like [starting+label]
e.g. [0x100 + main] for the code to work properly.

Virtual layout of 2 processes:
|--|--|--|--|     |--|--|--|--|--|--|
A1 A2 A3 A4       B1 B2 B3 B4 B5 B6

0								 									0xFFF
|---------------|--|---|--|--|---|--|--|--|--|----------------------|
                A4     B1 A3 A1  B2 A2 B3 B4                       end

Processes A & B always have separated address spaces, but splited in chunks called "frames"
Each "frame" correspond to a "page" in virtual address, they have the same size (4kB).
The contiguous virtual address space starting at 0 for each process in divided into pages that
are mapped into frames into physical memory.
The mapping is made page-wise (foreach page) and not address-wise (too long)

To calculate the mapping between pages and frames the MMU uses it's internal register:
the relocation register, that contains the address of the "page directory" for the current process.

Before diving into the computation of the adresses, let's look the structure of the page directory :

Note that because pages and frames must be aligned on 4KB boundaries (4KB being 0x1000 bytes),
the least significant 12 bits of the 32-bit adresses of pages/frames are always 0. The architecture uses them to store flags.

				PDE						PTE
CR3	-> *Page directory Entry	-> *Page Table Entry	-> Page (4kB)
								-> *Page Table Entry	-> Page (4kB)
								... (max 2^10)
	-> *Page Directory Entry
	... (max 2^10 => 1024)

Max size structure :
1024 PDE of 32 bits each pointing to 1024 PTE of 32 bits (each pointing to 4kib)
=> 1024 * 32 * 1024 * 32 => 32kbit * 32kbit => 1Gbit

CR3: Physical adress of the first page directory entry

Page Directory Entry:
 31 30 29 28 27 26 25 24 23 22 21 20 19 18 17 16 15 14 13 12 11 10  9  8  7  6  5  4  3  2  1  0 
| page table address 4kb aligned (max 0xfffff000)           |Avail   |G |S |0 |A |D |W |U |R |P |

 0  1  2  3  4  5  6  7  8  9  10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31
|P |R |U |W |D |A |0 |S |G |Avail   | page table 4kb aligned address (max 0xfffff000)           |

G - Ignored
S - Page size (0 for 4kb, 1 for 4 Mb)
A - Accessed
D - Cache disabled if the bit is set, the page will not be cached, otherwise it will be.
W - Write Through if the bit is set, write-through caching is enabled. If not, then write-back is enabled instead
U - User / Supervisor (page permission level, 0 for all, 1 for supervisor)
R - Read / Write (when set), read-only otherwise
P - Present

Page Table Entry:
 31 30 29 28 27 26 25 24 23 22 21 20 19 18 17 16 15 14 13 12 11 10  9  8  7  6  5  4  3  2  1  0 
| physical page adress (frame) 4kb aligned                  |Avail   |G |0 |D |A |C |W |U |R |P |

 0  1  2  3  4  5  6  7  8  9  10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31
|P |R |U |W |C |A |D |0 |G |Avail   | physical page adress (frame) 4kb aligned                  |

G - Global
D - Dirty
A - Acessed
C - Cache disabled
W - Write Through
U - User / Supervisor
R - Read / Write
P - Present

Virtual adress :

 31 30 29 28 27 26 25 24 23 22 21 20 19 18 17 16 15 14 13 12 11 10  9  8  7  6  5  4  3  2  1  0 
| page directory entry index  | page table entry index      | @ offset in page                  |

 0  1  2  3  4  5  6  7  8  9  10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31
|@ offset in page                   | page table entry index      | page directory entry index  |


## Translation virtual to physical :

((PTE *) ((PDE *) cr3 + sizeof(PDE) * (vaddr >> 22))->page_table + (sizeof(PTE) * (vaddr >> 12 & 0x3FF))->page_address + (vaddr & 0xFFF)
((char *) ((char *) ((char *) cr3) + sizeof(PDE) * (vaddr >> 22)) + (sizeof(PTE) * (vaddr >> 12 & 0x3FF)) + (vaddr >> 12 & 0x3FF) + (vaddr & 0xFFF)

([([cr3 + 32 * (vaddr >> 22)] >> 12) + (32 * (vaddr >> 12 & 0x3FF))] >> 12) + (vaddr & 0xFFF)

#define page_dir 0xFFF
#define page_table 0xFF5

u32 pd_index = (u32) vaddr >> 22; // get the page directory (bits 22 to 31)
u32 pt_index = (u32) vaddr >> 12 & 0x03FF; // get the page table index (bits 12 to 21)
					// 0x03FF => b001111111111 => first 10 bits

u32 *pd_entry = (u32 *) page_dir + (1024 * pd_index);
if (!pd_entry->P)
	pd_entry = new_pd_entry();

u32 *pt = (u32 *) page_table + (1024 * pd_index);

# MMU

# Paging

# Segmentation

# Virtual to Physical

# dev

const bits = vaddr => ('0'.repeat(32 - vaddr.toString(2).length) + vaddr.toString(2)).split('').reduce((a, e, i) => { a[i] = e|0; return a }, [])