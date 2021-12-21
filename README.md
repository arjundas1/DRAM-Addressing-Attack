<h1 align="center"> DRAM Addressing Attack </h1>

<p align="center">
  <a href="https://github.com/arjundas1/DRAM-Addressing-Attack">
    <img src="https://media.geeksforgeeks.org/wp-content/uploads/20200501212758/DRAM1.png" width="650" height="300">
  </a>
</p>

<details open="open">
  <summary>Table of Contents</summary>
  <ol>
    <li><a href="#introduction">Introduction</a></li>
    <li><a href="#project-team">Project Team</a></li>
    <li><a href="#project-objective">Project Objective</a></li>
    <li><a href="#background">Background</a></li>
    <li><a href="#implementation">Implementation</a></li>
    <li><a href="#conclusion">Conclusion</a></li>
    <li><a href="#references">References</a></li>
  </ol>
</details>

## Introduction

The main memory of computers consists of Dynamic Random-Access Memory
cells, that are organized in a hierarchy of channels, Dual Inline Memory Modules,
ranks and banks. A system has one or more channels, which are physical links
between DRAM modules and memory controllers. Multiple DIMM are
connected to each channel, and each DIMM has one or two ranks. Each rank has
8 or 16 banks, which in turn contain the actual memory arrays that are organized
in 215 rows and 210 columns. 

Two addresses can be only physically adjacent in
a DRAM chip if they are in same channel. The mapping from physical addresses
to the channels are completely dependent on the processor. Generally, a mapping
is done by a linear function using an XOR of address bits. This mapping is not
disclosed by manufacturers.

## Project Team
Guidance Professor: Dr. Yokesh Babu S, School of Computer Science and Engineering, VIT Vellore.

Team members:

|Sl.No. | Name  | Registration No. |
|-| ------------- |:-------------:|
|1|   Arjun Das      | 20BDS0129     |
|2| Nishanth RB      | 20BCE2406     |

## Project Objective

The purpose of this project is to explore cross-CPU
side channel attacks, thereby understanding its wide applications. Exploiting the
addressing of DRAM and the presence of its buffers in each bank of the DRAM
modules has been given prime focus. It is essential to gain information of the
mapping of the physical address of the DRAM bank, for which reverse
engineering methods have been discussed and implemented. Such mapping may
be helpful for speeding up a row hammer attack. 

The publicly available
information on the existing methods of DRAM exploitation have been studied
and built further. Reverse engineering, followed by verification of the mapping
have been implemented on the 9
th generation of Intel’s i7 processor.
Furthermore, a covert channel of communication has been established between
two processes that are running on the same machine, just like a cloud computing
environment.


## Background

The individual rows of a memory bank cannot be accessed directly. There is a
unique row buffer between every DRAM cell and memory bus, which stores the
entire row and acts as a directly mapped cache. Requests to addresses in the active
row is served directly from the buffer. If a different row needs to be accessed,
then the data of that row is copied to the row buffer and the request is serviced
thereby. On such a conflict, the time taken to access the data is more which allows
us to perform the addressing attack.

In order to exploit the row buffer, alternate access of elements from the DRAM
at two different physical addresses is made to be done and the average time taken
over several such accesses is measured. It needs to be ensured that the element
must be fetched from the main memory by flushing it from the cache before every
access. The average time will be high if two addresses happen to be mapped in
the same bank in the physical memory. The contrary is observed if two elements
belong to different banks.

A processor places spatially related elements in different banks so that repeated
access to these elements would be efficient. The mapping function for the 9th
generation of Intel’s i9 processor is unknown, but it is expected to provide log2(n)
bits where n is the total number of banks that exists in the DRAM. The semantic
meaning of the bits contains as: one bit identifies the channel, another identifies
the rank and two to identify the bank within the channel, etc. The physical voltage
measurement in the bus method is more efficient than the software based reverse 
engineering method as mentioned in Pessl, et al., but highly inconvenient at the
same time because it requires to physically open the machine. 

## Implementation

### Reverse Engineering

The program allocates an enormous portion of the DRAM (almost 70%) by
declaring a very large and initialising it, in order to prevent lazy allocation. The
pagemap of the process is used to find page tables and determine the virtual
address to physical address translation. The program then creates a set of large
numbers for random addresses, including virtual and physical. A random address
is selected from these and the set is partitioned into subsets in a way that the
average computation time of our reference element with every other element
many times. A histogram is yielded with access time on x-axis and fraction of
accesses on y-axis. 

The subset partition can be manually tweaked by setting a few
additional parameters that can algorithmically partition the set. The target number
of partitions is user defined, which is the product of the number of DIMMs,
channels, ranks and banks. At the end of this algorithm, the subset elements
belong to the same bank. If the set is a number that is in power of two, the XOR
function will give uniform distribution. The addresses can now be represented as
a linear equation system. The search space is not large and hence generation of
every possible function is possible. Iteration over all the bitmasks are done and
for each bitmask, every address in the set is applied. 

If the XOR of the bits is same for all the addresses of an individual set, then the function can be stored as
a possible solution. From these solutions, we remove all functions that are linear
combinations of other functions using Gauss-Jordan elimination method. To
eliminate the spurious functions that can be formed due to wrong computation 
and subset formation, we run the measurement multiple times and intersect the
final solution to get a high probability of the actual function.

### Building a Covert Channel

If any of the functions obtained give a different value then the addresses belong
to different banks. Two addresses are generated, one in both the processes, in
such a way that they are in the same bank but not in the same row. The receiving
process continuously accesses its address during an agreed time interval and then
calculates the average time taken to access that address using the same method as
used in reverse engineering of mappings. 

The sending process transfers messages in bits. The sending process also keeps accessing its address if it has to transfer a
bit during the time interval, but only after flushing the cache, else it sits idle. The
sender transmitting ‘1’ will show a higher average when compared to the average
time the sender takes to transmit ‘0’ due to higher number of row buffer misses.
The receiver compares the average time with the threshold time to determine if
the transferred bit was 0 or 1. 

## Conclusion

The final estimated functions that were found are:

- h0 = p6 ⊕ p13
- h1 = p14 ⊕ p17
- h2 = p15 ⊕ p18
- h3 = p16 ⊕ p19

Here, hi represents the ith bit used to determine the bank and pi represents the ith
bit of the physical address.

The covert channel’s development was successful when running on the same
machine. Messages were transmitted with very high reliability, and at the cost of 
slow transmission rate. The access time difference between sending a 1 and a 0
was large enough to enable us to completely remove random noise effect and get
100% reliability when the threshold was set appropriately. 

## References
- [_Peter Pessl, Daniel Gruss, Clementine Maurice, Michael Schwarz, and Stefan Mangard. DRAMA: exploiting DRAM addressing for cross-cpu attacks. In 25th USENIX Security Symposium, USENIX Security 16, Austin, TX, USA, August 10-12, 2016., pages 565–581, 2016._](https://github.com/arjundas1/DRAM-Addressing-Attack/blob/main/References/DRAMA-%20exploiting%20DRAM%20addressing%20for%20cross-cpu%20attacks.pdf)
- _Fangfei Liu, Yuval Yarom, Qian Ge, Gernot Heiser, and Ruby B. Lee. Last-level cache side-channel attacks are practical. In 2015 IEEE Symposium on Security and Privacy, SP 2015, San Jose, CA, USA, May 17-21, 2015, pages 605–622, 2015._
- _Yuval Yarom and Katrina E. Falkner. Flush+reload: a high resolution, low noise, L3 cache sidechannel attack. IACR Cryptology ePrint Archive, 2013:448, 2013._ 
- _Yoongu Kim, Ross Daly, Jeremie Kim, Chris Fallin, Ji-Hye Lee, Donghyuk Lee, Chris Wilkerson, Konrad Lai, and Onur Mutlu. Flipping bits in memory without accessing them: An experimental study of DRAM disturbance errors. In ACM/IEEE 41st International Symposium on Computer Architecture, ISCA 2014, Minneapolis, MN, USA, June 14-18, 2014, pages 361–372, 2014._
