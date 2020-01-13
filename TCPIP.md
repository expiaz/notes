# Link layer
## Ethernet frame
distinguish a machine on a network by MAC@
goes from a machine to another ID by MAC@ via direct cables

bytes
0          7     8                 14           20                 22        68                     72
| Preamble | SFD | MAC destination | MAC source | Ethertype/length | Payload | Frame check sequence |
7B         1B    6B                6B           2B                 46-1500B  4B                     

Preamble = alternating 0 & 1 (10101010 10101010 10101010 10101010 10101010 10101010 10101010)
SFD = Start frame delimiter (10101011)
Frame check sequence = 32 bit CRC
Ethertype = type of payload (0x0800 for IPv4, 0x0806 for ARP ...)

# Internet Layer
## IP Header
distinguish a network by IP@
goes from a network to another ID by IP@ via routers

0       4       8               16                            32
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Version|Length |Type of Service| Total Tength                |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0                               16    19                      32
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Identification                |Flags| Fragment offset       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0                                                             32
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Source IP address                                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0                                                             32
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Destination IP address                                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0                                                             32
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Options (optionnal)                                         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

Version = the version of the IP protocol. For IPv4, this field has a value of 4
Length = the length of the header in 32-bit words (usually 5 because options not needed)
Type of Service = type of the datagram behind (6 for TCP). The first 3 bits are the priority bits.
Total length = the length of the entire packet (header + data)

# Tranport Layer
## TCP Header
distinguish the server on a machine by port nb

0                               16                            32
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Source port                   | Destination port            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0                                                             32
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Sequence number                                             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0                                                             32
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Acknowledgment number (if ACK set)                          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0       4           10          16                            32
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Data  |           |U|A|P|R|S|F|                             |
| Offset| Reserved  |R|C|S|S|Y|I| Window                      |
|       |           |G|K|H|T|N|N|                             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0                               16                            32
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Checksum                      | Urgent Pointer              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0                                             23              32
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Options                                     | Padding       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0                                                             ?
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Data                                                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

# Application Layer
## HTTP


# Total frame

+----------------+-----------+------------+------+--------------+
| Ethernet frame | IP Header | TCP Header | HTTP | end of frame |
+----------------+-----------+------------+------+--------------+
^                ^           ^            ^      ^              ^
|                |           |            |      |              |
+---------------------------------------------------------------+
| Link Layer                                                    |
+---------------------------------------------------------------+
                 |           |            |      |
                 +-------------------------------+
                 | Internet Layer                |
                 +-------------------------------+
                             |            |      |
                             +-------------------+
                             | Transport Layer   |
                             +-------------------+
                                          |      |
                                          +------+
                                          | Application Layer
                                          +------+