UPDATE: Dave Anderson proposed an alternative algorithm on his blog
  http://da-data.blogspot.com/2014/03/a-public-review-of-cuckoo-cycle.html
that uses about (1+K)/64 times the memory at roughly K times slowdown.
I am (re-)implementing this "tomato" (his pronouncable spelling of tmto,
or time-memory trade-off) in branch "dave", soon to become the new
reference implementation. Cuckoo Cycle aims to be tmto-hard in the sense
of using less than one bit per nonce causing a 100-fold slowdown.
A $1000 bounty is offered for a hardness disproving implementation.

The README in branch dave updates items 1), 6), and 7) below.

cuckoo
======

Mining is generally considered to be inherently power hungry but it need not be.
It’s a consequence of making the proof of work computationally intensive.
If computation is minimized in favor of random access to gigabytes of memory
(incurring long latencies), then mining will require large investments in RAM
but relatively little power.

Cuckoo Cycle represents a breakthrough in three important ways:

1) it performs only one very cheap siphash computation for about 3.3 random accesses to memory,

2) its memory requirement can be set arbitrarily

3) verification of the proof of work is instant, requiring 2 sha256 and 42 siphash computations.

Runtime in Cuckoo Cycle is completely dominated by memory latency. It promotes the use
of commodity general-purpose hardware over custom designed single-purpose hardware.

Other features:

4) proofs take the form of a length 42 cycle in the Cuckoo graph.

5) it has a natural notion of (base) difficulty, namely the number of edges in the graph;
   above about 60% of size, a 42-cycle is almost guaranteed, but below 50% the probability
   starts to fall sharply.

6) running time for the current implementation on high end x86 is under 24s/GB single-threaded,
   and under 3s/GB for 12 threads.

7) making cuckoo use a significant fraction of the typical memory of a botnet computer
   will send it into swap-hell, and likely alert its owner.
