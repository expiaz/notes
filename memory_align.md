Why memory diagrams are from higher to lower bits (from intel doc) ?

If we look at bitwise operators and how memory is represented in computer :

natural notation:
bits	0  1  2  3  4  5  6  7
2^	0  1  2  3  4  5  6  7

computer notation:
bits	7  6  5  4  3  2  1  0
2^	7  6  5  4  3  2  1  0

Because binary number 1011 is 2^3x1 + 2^2x0 + 2^1x1 + 2^0x1 = 8 + 0 + 2 + 1 = 11
and not 2^0x1 + 2^1x0 + 2^2x1 + 2^3x1 = 1 + 0 + 4 + 8 = 13
So binary notation is written from right to left because the most significant bit is the last one
and less significant the first one

Bitwise operators (shifts) indicates us that :

when shifting bits to the right :
4 >> 1 = 0100 >> 1 = 0010 = 2 => it decreases the represented number by / 2

when shifting bits to the left :
4 << 1 = 0100 << 1 = 1000 = 8 => it increases the represented number by *2

So upper left most bit is the most significant and upper right the less significant

that's may be why diagrams are drawn from right to left bit order, to not confuse the reader
during bitwise operations for flags (and bc it's real representation on the machine ?)