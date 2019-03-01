# Chapter 2

## 2.1 Information Storage

A machine-level program veiws memory as a very large array of bytes, referred to as virtual memory. Every byte of memory is identified by a unique number, known as address.

### 2.1.3 Addressing and Byte ordering

In virtually all machines, a multi-byte oject is stored as a contiguous sequence of bytes, with the address of the object given by the smallest address of the bytes used. For example , suppose a variable x of type int has address 0x100; Then the 4 bytes of x would be stored in memory locations 0x100, 0x101, 0x102, 0x103.

There are two byte orderings. *little endian* is the least significant byte comes first, and *big endian* is the most significant byte comes first. Note that in the word 0x01234567 the high-order byte is 0x01, while the lower-order byte is 0x67.

Most Intel-compatible machine operate exclusively in little-endian mode. And byte ordering becomes fixed once a particular operating system is chosen.

### 2.1.9 Shift Operations in C

Generally, machine support two forms of right shift:
+ Logical. A logical right shift fills the left end with k zeros.
+ Arithmetic. An arithmetic right shift fills the left end with k repetitions of the most significant bit.

## 2.4 Floating-Point

The IEEE floating-point standard represents a number in a form V = \( (-1)^{s} \times M \times 2^{E} \):
+ The *sign s* determines whether the number is negative or not.
+ The *significand M* is a fractional binary number that ranges either between 1 and 2 or between 0 and 1.
+ The *exponent E* weights the value by a power of 2

In the single-precison floating-point format, fields s, exp and frac are 1, 8 and 23 bits each. In the double-precision floating-point format, fields s, exp and fraca re 1, 11 and 52 for each. Usually, the error in represent floating-point number could be raised by the limitation of frac or sometimes it's just binary cannot represent specific fractions in decimal.

# Chapter 3

## 3.2 Program Encodings

`linux> gcc -Og -o p p1.c p2.c`

The command `gcc` indicates the gcc C compiler. The command-line option `-Og` instructs the compiler to apply a level of optimization that yields machine code that follows the overall structure of the original C code. In practice, higher levels of optimization (e.g., specified with the option `-O1` or `-O2`) are considered a better choice in terms of the resulting program performance.

First, the C preprocessor expands the source code to include any files specified with `#include` commands and to expand any macros, specified with #define declarations. Second, the compiler generates assembly-code versions of the two source files having names p1.s and p2.s. Next, the assembler converts the assembly code into binary object-code files p1.o and p2.o. Object code is one form of machine code-it contains binary representations of all of the instructions, but the addresses of global values are not yet filled in. Finally, the linker merges these two object-code files along with code implementing library functions (e.g., printf) and generates the final executable code file p.

### 3.2.1 Machine-level Code

The format and behavior of a machine-level program is defined by the *instruction set architecture*, or ISA, defining the processor state, the format of the insturctions, and the effect each of these instructions will have on the state.

+ The program counter (commonly referred to as the PC, and called %rip in x86-64) indicates the address in memory of the next instruction to be executed.
+ The integer register file contains 16 named locations stroing 64-bit values.
+ The condition code registers hold status information about the most recently executed arithmetic or logical instruction. These are used to implement conditional changes in the control or data flow.
+ A set of vector registers can each hold one or more integer or floating-point values.

Whereas C provides a model in which objects of different data type can be declared and allocated in memory, machine code view the memory as simply a large byte-addressable array. Aggregate data types in C such as arrays and structures are represented in machine code as contiguous collections of bytes. Even for scalar data types,  assembly code makes no distinctions between signed or unsigned integers, between different types of pointers, or even between pointers and integers.

The porgram memory contains the executable machine code for the program, some information required by the operating system, a run-time stack for managing procedure calls and returns, and blocks of memory allocated by the user.

`gcc -S` will cause gcc to run the compiler, generating an assembly file and go no further. If we use the `-c` command-line option, gcc will both compile and assemble the code, generating an object-code file.

Generating the actual executable code requires running a linker on the ser of object-code files, one of which must contain a function main. One task for the linker is to match function calls with the locations of the executable code for those functions.

> **Aside** ATT versus Intel assembly-code formats  
We see the the Intel and ATT formats differ in the following ways:  
> + The Intel code omits the size designation suffixes.
> + The Intel code omits the % character in fromt of the register names
> + Intel code has a different way of describing locations in memory-for example, QWORD PTR[rbx] rather than (%rbx).
> + Instruction with multiple operands list them in the reverse order. `ATT: src, dst`. `Intel: dst, src`.

# 3.3 Data Formats

Due to tis origins as a 16-bit architecture that expanded into a 32-bit one, Intel use the term "word" to refer to a 16-bit data type. Based on this, they refer to 32-bit quantities as "double words," and 64-bit quantities as "quad words." Standard int values are stored as double words (32 bits). Pointers are stored as 8-byte quad words, as would be expected in a 64-bit machine. With x86-64, data type *long* is implemented with 64 bits, allowing a very wide range of values.


## 3.4 Accessing Information

An x86-64 central processing unit contains a set of 16 *general-purpose registers* stroing 64-bit values. These registers are used to store integer data as well as pointers.

When these instructions have registers as destinations, two conventions arise for what happens to the remaining bytes in the register for instructions that generate less than 8 bytes: Those that generate 1- or 2-byte quantities leave the remaining bytes unchanged. **Those that generate 4-byte quantities set the upper 4 bytes of the register to zero.** The latter convention was adopted as part of the expansion from IA32 to x86-64.

Stack pointer, `%rsp`, is used to indicate the end position in the run-time stack.

### 3.4.1 Operand Specifiers

Source vaules can be given as constants or read from registers or memory. Results can be stored in either registers or memory. Thus, the different operand possibilities can be classified into three types. The first type, *immediate*, is for constant values. In ATT-format assembly code, these are written with a `$` followed by an integer using standard C notation. The second type, *register*, denotes the contents of a register. The third type of operand is a *memory* reference, in which we access some memory location according to a computed address, often called the *effective address*.

### 3.4.2 Data Movement Instructions

x86-64 imposes the restriction that a move instruction cannot have both operands refer to memory locations. Copying a value from one memory location to anothe would requires load and store.

When `movl` has a register as the destination, it will also set the high-order 4 bytes of register to 0. This arises from the convention adopted in x86-64 as metioned in 3.4.

The `cltq` instruction has no operands--it always uses register %eax as its source and %rax as the destination for the sign-extended result. It therefore has the exact same effect as the instruction movslq %eax, %rax, but it has a more compact encoding.

### 3.4.4 Pushing and Popping Stack Data

The stack grons downward such that the top element has the lowest address of all stack elements. The stack pointer %rsp holds the address of the top stack element.

Pushing a quad word value onto the stack involves first decrementing the stack pointer by 8 and then writing the value at the new top-of-stack address.

## 3.5 Arithmetic and Logical Operations

### 3.5.1 Load Effective Address

The *load effective address* instruction `leaq` is actually a variant of the `movq` instruction. It has the form of an instruction that reads from memory to a register but it does not reference memory at all. It first operand appears to be a memory reference, but instead of reading from the designated location, the instruction copies the effective address to the destination. It can be used to compactly describe common arithmetic operations. For example, if register %rdx contains value `x`, then the instruction `leaq 7(%rdx, %rdx, 4), %rax` will set register `%rax` to 5x + 7. Compilers often find clever uses of `leaq` that have nothing to do with effective address computations. The destination operand must be a register.

### 3.5.2 Unary and Binary Operations

As with binary operations, the first operand can be either an immediate value, a register, or a memory location. The second can be either a register or a memory location. As with the mov instruction, the two operands cannot both be the memory location. Note that when the second operand is a memory location, the processor must read the value from memory, perform the operation, and then write the result back to memory.

### 3.5.3 Shift operations

The different shift instructions can specify the shift amount either as an immediate value or with the single-byte register %cl. With x86-64, a shift instruction operating on data values that are w bits long determines the shift amount from the low-order m bits of register %cl, where 2^m = w. The high-order bits are ignored. So, for example, when register %cl has hexadecimal value 0xFF, then instruction `salb` would shift by 7, while `salw` would shift by 15, `sall` would shift by 31, and `salq` would shift by 63. The destination operand of a shift operation can be either a register or a memory location.

### 3.5.5 Special Arithmetic Operations

The x86-64 instruction set includes two different "one-operand" multiply insturctions to compute the full 128-bit product of two 64-bit values, one for unsigned (mulq) and one for two's-complement (imulq) multiplication. For both of these instructions, one argument must be in register %rax, and the other is given as the instruction source operand. The product is then stored in register %rdx (high-order 64 bits) and %rax (low-order 64 bits).

The signed division instruction `idivl` takes as its dividend the 128-bit quantity given as the instruction operand. The instruction stores the quotient in register %rax and the remainder in register %rdx.

## 3.6 Control

Machine code provides two basic low-level mechnisms for implementing conditional behavior: it tests data values and then alters either the control flow or the data flow based on the result of these tests.

### 3.6.1 Condition Codes

In addition to the integer registers, the CPU maintains a set of single-bit *condition code* registers describing attributes of the most recent arithmetic or logical operation. These registers can then be tested to perform conditional branches.

+ CF: Carry Flag. The most recent operations generated a carry out of the most significant bit. Used to detect overflow for unsigned operations.
+ ZF: Zero Flag. The most recent op yielded zero.
+ SF: Sign Flag. The most recent op yielded a negative value.
+ OF: Overflow Flag. The most recent op caused a two's-complement overflow--either negative or positive.

The `leaq` instruction does not alter any condition codes, since it is intended to be used in address computation.

### 3.6.6 Conditional Moves

The conventional way to implement conditional operations is through a conditional transfer of control, where the program follows one execution path when a condition holds and nother when it does not. This mechanism is simple and general, but it can be very inefficient on modern processors.

An alternate startegy is through a conditional transfer of data. This approach computes both outcomes of a conditional operations and then selects one based on whether or not the condition holds. This strategy makes sense only in restricted cases, but it can then be implemented by a simple conditional move instruction that is better matched to the performance characteristics of modern processors.

With conditional jumps, when branch is hard to predict, it will perform bad. Mainly caused by pipelining and the cost of misprediction. While conditional moves are faster in average because the flow of contol does not depend on data. 


## 3.7 Procedures



# Chapter 6
## 6.2 Locality
Locality is typically described as having two distinct forms: *temporal locality* and *spatial locality*. In a program with good temporal locality, a memory location that is referenced once is likely to be referenced again multiple times in the near future. In a program with good spatial locality, if a memory location is referenced once, then the program is likely to referenced a nearby memory location in the near future.

In general, programs with good locality run faster than programs with poor locality. All levels of modern computer systems, from the hardware, to the operating system, to application programs, are designed to exploit locality. At the hardware level, the principle of locality allows computer designers to speed up main memory accesses by introducing small fast memories known as *cache memories* that holds blocks of the most recently referenced instructions and data items. At the operating system level, the principle of locality allows the system to use the main memory as a cache of the most recently referenced chunks of the virtual address space. Similarly, the operating system uses main memory to cache the most recently used disk blocks in the disk file system.

### 6.2.1 Locality of References to Program Data
A function that visits each element of a vector sequentially is said to have a *stride-1 reference pattern*. Visiting every k-th element of a continguous vector is called a *stride-k reference pattern*. Stide-1 reference patterns are a common and important source of spatial locality in programs. In general, as the stride increase, the spatial locality decreases.


### 6.2.3 Summary of Locality
+ Programs that repeatedly reference the same variables enjoy good temporal locality,
+ For programs with stride-k reference patterns, the smaller the stride, the betther the spatial locality.
+ Loops have good temporal and spatial locality with respect to instruction fetches. The smaller the loop body and the greater the number of loop iterations, the better the locality.

## 6.3 The memory hierarchy

### 6.3.1 Caching in the Memory Hierarchy
The central idea of a memory hierarchy is that for each k, the faster and smaller storage device at level k serves as a cache for the larger and slower storage device at level k+1.

Data are always copied back and forth between level k and level k+1 in block-size transfer unit. It's important to realize that while the block size is fixed between any particular pair of adjacent levels in the hierarchy, other pairs of levels can have different block sizes.

#### Cache Hits
The program reads directly from level k when hit.

#### Cache Misses
Whe there is a miss, the cache at level k fetches the block containing d from the cache at level k+1, possibly overwritting an existing block if the level k cache is full.

The process of overwritting an existing block is known as *replacing* or *evicting*. The block that is evicted is sometimes referred to as a *victim block*. The decision about which block to replace is governed by the cache's *replacement policy*.

After the cache at level k has fetched the block from level k+1, the program can read from level k as before.

#### Kinds of Cache Misses
An empty cache is sometimes referred to as a *cold cache*, and misses of this kind are called *compulsory misses* or *cold misses*. Cold misses are important because they are often transient events that might not occur in steady state, after the cache has been *warmed up* by repeated memory accesses.

Hardware caches typically implement a simpler placement policy that restricts a particular block at level k+1 to a small subset of the blocks at level k.

Restrictive placement policies of this kind lead to a type of miss known as a *conflict miss*, in which the cache is large enough to hold the referenced data objects, but because they map to the same cache block, the cache keeps missing.

When the size of the working set exceeds the size of the cache, the cache will experience what are know as *capacity misses*. In other words, the cache is just too small to handle this particular working set.

### 6.3.2 Summary of Memory Hierarchy Concepts
+ *Exploiting temporal locality*.  Because of temporal locality, the same data objects are likely to be reused multiple times. Once a data object has been copied into the cache on the first miss, we can expect a number of subsequent hits on that object.
+ *Exploiting spatial locality*. Blocks usually contain multiple data objects. Because of spatial locality, we can expect that the cost of copying a block after a miss will be amortized by subsequent references to other objects within that block.

## 6.4 Cache Memories
Consider a computer system where each memory address has m bits that form M = 2^m unique address. A cache is organized as an array of S=2^s *cache sets*. Each et consists of E *cache lines*. Each line consists of a data block of B=2^b bytes, a valid bit that indicates whether or not the line contains meaningful information, and t=m-(b+s) *tag bits* that uniquely identify the block stored in the cache line. The size of a cache, C,would be S * E * B.

When the CPU is instructed by a load instruction to read a word from address A of main memory, it sends address A to the cache. If the cache is holding a copy of the word at address A, it sends the word immediately back to the CPU. The parameters S and B induce a partitioning of the m address bits into the three fields tag, set index, and block offset. The set index bits tell us which set the word must be stored in. Once we know which set the word must be contained in, the *t* tag bits in A tell us which line in the set contains the word. A line in the set contains the word iff the valid bit is set and the tag bits in the line match the tag bits in the address A. Once we have located the line identified by the tag in the set identified by the set index, then the *b block offset* bits give us the offset of the word in the B-byte data block.

### 6.4.2 Direct-Mapped Caches
Caches are grouped into different classes based on E, the number of cache lines per set. A cache is exactly one line per set (E=1) is known as a *direct-mapped* cache. The process that a cache goes through of determining whether a request is a hit or a miss and then extracting the requested word consists of three steps: (1) set selection, (2) line matching, and (3) word extraction.

#### Set selection in Direct-Mapped Caches
In this step, the cache extracts the *s* set index bits from the middle of the address for w.

#### Line Matching in Direct-Mapped Caches
A copy of *w* is contained in the line iff the valid bit is set  and the tag in the cache line matches the tag in the address of w. If either the valid bit was not set or the tag did not match, then we would have had a cache miss.

#### Word Selection in Direct-Mapped Caches
The block offset bits provide us with the offset of the first byte in the desired word.

#### Line Replacement on Misses in Direct-Mapped Caches
For a direct-mapped cache, where each set contains exactly one line, the replacement policy is trivial: the current line is replaced by the newly fetched line.

#### Putting it Together: A Direct-Mapped Cache in Action
There are some interesting things to notice about this enumerated space:
+ The concatenation of the tag and index bits uniquely identifies each block in memory.
+ Multiple blocks in memory may map to the same cache set.
+ Blocks that map to the same cache set are uniquely identified by the tag.

#### Conflict Misses in Direct-Mapped Caches
COnflict misses in direct-mapped caches typically occur when programs access arrays whose sizes are a power of 2. One easy solution is to put B bytes of padding at the end of each array and they would map to different sets.

#### Why index with the middle bits?
If the high-order bits are used as set index, then some continguous memory blocks will map to the same cache set. If a program has good spatial locality and scans the elements of an array sequentially, then the cache can only hold a block-size chunk of the array at any point in time. This is an inefficient use of the cache. Contrast this with middle-bit indexing, where adjacent blocks always map to different cache sets.

### 6.4.3 Set Associative Caches
A cache with 1 <  E < C/B is often called an E-way set associative cache. 

#### Set selection in Set Associative Caches
Identical to a direct-mapped cache.

#### Line Matching and Word Selection in Set Associative Caches
An important idea here is that any line in the set can contain any of the meory blocks that map to that set. So the cache must search each line in the set for a valid line whose tag matches the tag in the address. Word selection is the same as direct-mapped cache.

#### Line Replacement on Misses in Set Associative Caches
When we have a cache miss, the cache must fetch the block that contains the word from memory. However, once the cache has retrived the block, which line should it replace? Of course, if there is an empty line, then it should be a good candidate. But if there are no empty lines in the set, then we must choose one of the nonempty lines and hope that the CPU does not reference the replaced line anytime soon.

The simplest replacement policy is to choose the line to replace at random. Other more sophisticated policies try to minimize the probability that the replaced line will be referenced in the near future. A *least frequently used(LFU)* policy will replace the line that has been referenced the fewest times over some past time window. A *least recently used(LRU)* policy will replace the line that was last accessed the furthest in the past.

### 6.4.4 Fully Associative Caches
A *fully associative cache* consists of a single set (i.e., E=C/B) that contains all of the cache lines. Fully Associative Caches parse address into two fields, tag bits and block offset, because there is only one set.

#### Set Selection in Fully Associative Caches
Set selection here is trivial because there is only one set.

#### Line Matching and Word Selection in Fully Associative Caches
Identical to set associative cache. It is difficult and expensive to build an associative cache that is both large and fat. As a result, fully associative caches are only appropriate for small caches, such as the translation lookaside buffers(TLBs) in virtual memory systems that cache page table entries.

### 6.4.5 Issues with Writes
AFter the cache updates its copy of *w*, what does it do about updating the copy of *w* in the next level of the hierarchy? The simplest appraoch, known as *write-through*, is to immediately write w's cache block to the next lower level. This could cause bus traffic with every write. Another approach, know as *write-back*, defers the update as long as possible by writing the updated block to the next lower level only when it is evicted from the cache by the replacement algorithm. Because of locality, write-back can significantly reduce the amount of bus traffic, but it has the disadvantage of additional complexity. The cache must maintain an additional *dirty bit* for each cache line that indicates whether or not the cache block has been modified.

Another issue is how to deal with write misses. One approach, known as *write-allocate*, loads the corresponding block from the next lower level into the cache and the updates the cache block. Write-allocate tries to exploit spatial locality of writes but it has th disadvantage that every write miss results in a block transfer from the next lower level to the cache. An alternative, known as *no-write-allocate*, bypasses the cache and writes the word directly to the next lower level. write-through caches are typically no-write-allocate. Write-back caches are typically write-allocate.

In summary, for write-through, when the write his the data in cache, it would update the cache block and immediately write that cache block into the next lower level. When the write miss, which means the destination block is not in the cache, it would directly write into the next lower level. But for write-back, when the write hits the data in cache, it would update the cache block and defer the update of next lower level as long as possible by writing to the next level only when that block is evicted from the cache. When the write miss, it would load that block from the next lower level into cache and then updates the cache block while defer its update to the next lower level as long as possible.

As a rule, caches at lower levels of the memory hierarchy are more likely to use write-back instead of write-through because of the larger transfer times.

### 6.4.6 Anatomy of a Real Cache Hierarchy
A cache that holds instructions only is called an *i-cache*. A cache that holds program data only is called a *d-cache*. A cache holds both instructions and data is known as a *unified cache*. I-caches are typically read-only.

### 6.4.7 Performance Impact of Cache Parameters

#### Impact of Cache Size
On the one hand, a larger cache will tend to increase the hit rate. On the other hand, it is always harder to make large memories run faster. As a result, larger caches tend to increase the hit time.

#### Impact of Block Size
On the one hand, larger blocks can help incrase the hit rate by exploiting any spatial locality  that might exist in a program. However, for a given cache size, larger blocks imply a smaller number of cache lines, which can hurt the hit rate in programs with more temporal locality than spatial locality. Larger blocks also have a negative impact on the miss penalty.

#### Impact of Associativity
The advantage of higher associativity is that it decreases the vulnerability of the cache to thrashing due to conflict misses. However, higher associativity is expensive to implement and hard to make fast. It requires more tag bits per line, additional LRU state bits per line, and additional control logic. Higher associativity can increase hit time, because of the increased complexity, and it can also increase the miss penalty because of the increased complexity of choosing a victim line.

Traditionally, high-performance systems that pushed the clock rates would opt for smaller associativity for L1 caches and a higher degree of associativity for the lower levels, where the miss penalty is higher. For example, in Intel Core i7 system, the L1 and L2 caches are 8-way associative, and the L3 cache is 16-way.

#### Impact of Write Strategy
Write-throught is simpler to implement and the read misses are less expensive because they do not tirgger a memory write. On the other hand, write-back caches result in fewer transfers, which allows more bandwidth to memory for I/O devices that perform DMA. In general, caches further down the hierarchy are more likely to use write-back than write-through because the increasing transfer time.

# Chapter 8
Modern systems react to these situations by making abrupt changes in the control flow. In general, we refer to these abrupt changes as *exceptional control flow(ECF)*. ECF occurs at all levels of a computer system. For example, at the hardware level, events detected by the hardware trigger abrupt control transfers to exception handlers. At the operating systems level, the kernel transfers control from one user process to another via context switches. At the application level, a process can send a *signal* to another process that abruptly transfers control to a signal handler in the recipient.