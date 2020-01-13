### PIC & slave PIC (Programmable Interrupt Controller)

Basically, when a key is pressed, the keyboard controller tells a device called the Programmable Interrupt Controller, or PIC, to cause an interrupt.
Because of the wiring of keyboard and PIC, IRQ #1 is the keyboard interrupt, so when a key is pressed, IRQ 1 is sent to the PIC.
The role of the PIC will be to decide whether the CPU should be immediately notified of that IRQ or not and to translate the IRQ number into an interrupt vector (i.e. a number between 0 and 255) for the CPU's table.

The OS is supposed to handle the interrupt by talking to the keyboard, via in and out instructions, asking what key was pressed, doing something about it (such as displaying the key on the screen, and notifying the current application that a key has been pressed),
and returning to whatever code was executing when the interrupt came in.
Indeed, failure to read the key from the buffer will prevent any subsequent IRQs from the keyboard.

There are actually two PICs on most systems, and each has 8 different inputs, plus one output signal that's used to tell the CPU that an IRQ occurred. The slave PIC's output signal is connected to the master PIC's third input (input #2); so when the slave PIC wants to tell the CPU an interrupt occurred it actually tells the master PIC, and the master PIC tells the CPU. This is called "cascade". The master PIC's third input is configured for this and not configured as a normal IRQ, which means that IRQ 2 can't happen.

A device sends a PIC chip an interrupt, and the PIC tells the CPU an interrupt occurred.
When the CPU acknowledges the "interrupt occurred" signal, the PIC chip sends the interrupt number
(between 00h and FFh, or 0 and 255 decimal) to the CPU. When the system first starts up,
IRQs 0 to 7 are set to interrupts 08h to 0Fh, and IRQs 8 to 15 are set to interrupts 70h to 77h.
Therefore, for IRQ 6 the PIC would tell the CPU to service INT 0Eh, which presumably has code for interacting with whatever
device is connected to the master PIC chip's "input #6".
Note that interrupts are handled by priority level: 
- IRQ0 -> 0
- IRQ1 -> 1
- IRQ2 -> 2
- IRQ3 -> 8
- IRQ4 -> 9
- IRQ5 -> 10
- IRQ6 -> 11
- IRQ7 -> 12
- IRQ8 -> 13
- IRQ9 -> 14
- IRQ10 -> 15
- IRQ11 -> 3
- IRQ12 -> 4
- IRQ13 -> 5
- IRQ14 -> 6
- IRQ15 -> 7
So, if IRQ 8 and IRQ 3 come in simultaneously, IRQ 8 is sent to the CPU.
When the CPU finishes handling the interrupt, it tells the PIC that it's OK to resume sending interrupts:

mov al,20h // 20h is ack message
out 20h, al // 20h is address of master PIC

or for slave PIC :
mov al, 20h
out A0h, al // A0h is address of slave PIC
out 20h, al

then the PIC sends the interrupt assigned to IRQ 3, which the CPU handles (using the IDT to look up the handler for that interrupt).

Alert readers will notice that the CPU has reserved interrupts 0-31,
yet IRQs 0-7 are set to interrupts 08-0Fh.
Now the reserved interrupts are called when, for example,
a dreadful error has occurred that the OS must handle.
Now when the computer first starts up, most errors of this type won't occur.
However, when you enter protected mode (and every OS should use protected mode, real mode is obsolete),
these errors may occur at any time, and the OS needs to be able to handle them.
How's the OS going to tell the difference between
- INT 9, Exception: Coprocessor segment overrun
- INT 9: IRQ 1
Well, it can ask the device whether there is really an interrupt for that device.
But this is slow, and hackish, and not all devices are able to do this type of thing.
The best way to do it is to tell the PIC to map the IRQs to different interrupts, such as INT 78h-7Fh.
Note that IRQs can only be mapped to INTs that are multiples of 08h: 00h-07h, 08h-0Fh, 10h-17h, 17h-1Fh.
And you probably want to use 20h-27h, or greater, since 00h-1Fh are reserved by the CPU.
Also, each PIC has to be programmed separately. You can tell the Master PIC to map IRQs 0-7 to INTs 20h-27h,
but IRQs 8-F will still be INTs 70h-77h, unless you tell the Slave PIC to put them elsewhere as well.


### 8259 PIC

The 8259 PIC uses 2*8 pins chips to handle up to 15 interrupts.
That's why there is a master and slave PIC, the slave is connected to a master's PIN used not
to handle interrupts but to communicate with the slave.
The slave is connected by IRQ2 (pin 2) to the master, and master to the interrupt line.

CPU    |-----------------------------------------
       |
line   |----------------------------------------|
						|
master -IRQ1-IRQ2-IRQ3-IRQ4-IRQ5-IRQ6-IRQ7-IRQ8-|
	       |
       |--------
slave  |-IRQ9-IRQ10-IRQ11-IRQ12-IRQ13-IRQ14-IRQ15-

#### Reprogramming

Each chip have a command and data port :
master	- command:	0x20
	- data:		0x21
slave	- command:	0xA0
	- data:		0xA1

/!\/!\/!\
- Each PIC vector offset must be divisible by 8, as the 8259A uses the lower 3 bits for the interrupt number of a particular interrupt (0..7).
- The only way to change the vector offsets used by the 8259 PIC is to re-initialize it, which explains why the code is "so long" and plenty of things that have apparently no reasons to be here.
- If you plan to return to real mode from protected mode (for any purpose), you really must restore the PIC to its former configuration.

##### IRQs in Real Mode

Master PIC are IRQs 0 to 7 from 0x08 (offset) to 0x0F (8 interrupts)
Slave PIC are IRQs 8 to 15 from 0x70 to 0x77

##### Protected mode

End of Interrupt (EOI), used to signal the end of an IRQ based routine

if from slave:
	out 0xA0, 0x20
for all interrupts:
	out 0x20, 0x20

Reinitalisation of PIC

	// remaps IRQs 0-7 (master) to INT 78h-7Fh
	// and IRQs 8-F (slave) to INT 70h-77h

	in master_mask, 0x21 // save the masks
	in slave_mask, 0xA1

	out 0x20, 0x10 | 0x01 => 0x11 // start the ini process in cascade mode
	out 0xA0, 0x11

	out 0x21, 0x78 // ICW2: Master PIC vector offset
	out 0xA1, 0x70 // ICW2: Slave PIC vector offset

	out 0x21, 0x04 // ICW3: tell master that there is slave at IRQ2 (000 0100)
	
	out 0xA1, 0x02 // ICW3: tell slave PIC it is cascade identity (0000 0010)

	out 0x21, 0x01 // ICW4: 8006/88 (MCS-80/85) mode
	out 0xA1, 0x01

	out 0x21, master_mask // restore masks
	out 0xA1, slave_mask

Disabling the PIC

If you are going to use the processor local APIC and the IOAPIC, you must first disable the PIC. This is done via:

mov al, 0xff
out 0xA1, al
out 0x21, al

Masking PIC (out 0x21/0xA1, IRQnumber)

The PIC has an internal register called the IMR, or the Interrupt Mask Register.
It is 8 bits wide. This register is a bitmap of the request lines going into the PIC.
When a bit is set, the PIC ignores the request and continues normal operation.
Note that setting the mask on a higher request line will not affect a lower line.
Masking IRQ2 will cause the Slave PIC to stop raising IRQs.

Spurious IRQs

When an IRQ occurs, the PIC chip tells the CPU (via. the PIC's INTR line) that there's an interrupt, and the CPU acknowledges this and waits for the PIC to send the interrupt vector. This creates a race condition: if the IRQ disappears after the PIC has told the CPU there's an interrupt but before the PIC has sent the interrupt vector to the CPU, then the CPU will be waiting for the PIC to tell it which interrupt vector but the PIC won't have a valid interrupt vector to tell the CPU.

To get around this, the PIC tells the CPU a fake interrupt number. This is a spurious IRQ. The fake interrupt number is the lowest priority interrupt number for the corresponding PIC chip (IRQ 7 for the master PIC, and IRQ 15 for the slave PIC).

There are several reasons for the interrupt to disappear. In my experience the most common reason is software sending an EOI at the wrong time. Other reasons include noise on IRQ lines (or the INTR line).

Handling Spurious IRQs
For a spurious IRQ, there is no real IRQ and the PIC chip's ISR (In Service Register) flag for the corresponding IRQ will not be set. This means that the interrupt handler must not send an EOI back to the PIC to reset the ISR flag.

The correct way to handle an IRQ 7 is to first check the master PIC chip's ISR to see if the IRQ is a spurious IRQ or a real IRQ. If it is a real IRQ then it is treated the same as any other real IRQ. If it is a spurious IRQ then you ignore it (and do not send the EOI).

The correct way to handle an IRQ 15 is similar, but a little trickier due to the interaction between the slave PIC and the master PIC. First check the slave PIC chip's ISR to see if the IRQ is a spurious IRQ or a real IRQ. If it is a real IRQ then it is treated the same as any other real IRQ. If it's a spurious IRQ then don't send the EOI to the slave PIC; however you will still need to send the EOI to the master PIC because the master PIC itself won't know that it was a spurious IRQ from the slave.

## Mapping

IRQ	Description
0	Programmable Interrupt Timer Interrupt
1	Keyboard Interrupt
2	Cascade (used internally by the two PICs. never raised)
3	COM2 (if enabled)
4	COM1 (if enabled)
5	LPT2 (if enabled)
6	Floppy Disk
7	LPT1 / Unreliable "spurious" interrupt (usually)
8	CMOS real-time clock (if enabled)
9	Free for peripherals / legacy SCSI / NIC
10	Free for peripherals / SCSI / NIC
11	Free for peripherals / SCSI / NIC
12	PS2 Mouse
13	FPU / Coprocessor / Inter-processor
14	Primary ATA Hard Disk
15	Secondary ATA Hard Disk