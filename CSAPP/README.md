# Chapter 2

## 2.1 Information Storage
A machine-level program veiws memory as a very large array of bytes, referred to as virtual memory. Every byte of memory is identified by a unique number, known as address.
### Addressing and Byte ordering
In virtually all machines, a multi-byte oject is stored as a contiguous sequence of bytes, with the address of the object given by the smallest address of the bytes used. For example , suppose a variable x of type int has address 0x100; Then the 4 bytes of x would be stored in memory locations 0x100, 0x101, 0x102, 0x103.

There are two byte orderings. *little endian* is the least significant byte comes first, and *big endian* is the most significant byte comes first. Note that in the word 0x01234567 the high-order byte is 0x01, while the lower-order byte is 0x67.

Most Intel-compatible machine operate exclusively in little-endian mode. And byte ordering becomes fixed once a particular operating system is chosen.