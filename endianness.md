Big-endian

Most Significant Byte (MSB) containing the MSBit is stored first

	0x123456
0x0	-----
	|12
	|34
	|56
	|
0x100	-----

Used in network protocol (TCP/UDP, IPV4/6)

Little-endian

Less Significant Byte (LSB) is stored first

	0x123456
0x0	-----
	|56
	|34
	|12
	|
0x100	-----

/!\ reading a stored 16 bits value as a 32 bits (or less as more)

16 bits values 0x0A0B and 0x0C0D layout byte by byte in little endian:

|0x0B|0x0A|0x0D|0x0C|

read as 32 bits:

0x0C0D0A0B != 0xA0B0C0D