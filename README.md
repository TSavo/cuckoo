whitepaper: https://github.com/tromp/cuckoo/blob/master/cuckoo.pdf?raw=true

Cuckoo Cycle
============

Cuckoo Cycle is the first graph-theoretic proof-of-work.
Keyed hash functions define arbitrarily large random graphs,
in which certain fixed-size subgraphs occur with suitably small probability.
Asking for a cycle, or a clique, is a close analogue of asking for
a chain or cluster of primes numbers, which were adopted as the
number-theoretic proof-of-work in Primecoin and Riecoin, respectively.
The latest implementaton incorporates a huge memory savings proposed by Dave Andersen in

http://da-data.blogspot.com/2014/03/a-public-review-of-cuckoo-cycle.html


Cuckoo Cycle represents a breakthrough in three important ways:

1) it performs only one very cheap siphash computation for each random access to memory,

2) (intended) memory usage grows linearly with graph size, which can be set arbitrarily.
   there may be very limited opportunity to reduce memory usage without undue slowdown.

3) verification of the proof of work is instant, requiring 2 sha256 and 42x2 siphash computations.

Runtime in Cuckoo Cycle is dominated by memory latency (67%).
Any ASIC developed for Cuckoo Cycle will need to rely on commodity DRAM chips
(SRAM, while an order of magnitude faster, is also 2 orders of magnitude more expensive,
 and thus not competitive).
Such an ASIC would be focussed on orchestrating all the memory accesses which form the bottleneck,
and thus replacable by a programmable part (or FPGA) without much loss in efficiency.
In this way Cuckoo Cycle limits the advantage of custom designed single-purpose hardware
over commodity general-purpose hardware.

Other features:

4) proofs take the form of a length 42 cycle in the Cuckoo graph.

5) it has a natural notion of (base) difficulty, namely the number of edges in the graph;
   above about 60% of size, a 42-cycle is almost guaranteed, but below 50% the probability
   starts to fall sharply.

6) running time on high end x86 is 9min/GB single-threaded, and 1min/GB for 20 threads.


On July 23 2014, I posted the following message on https://bitcointalk.org/index.php?topic=707879.0

Cuckoo Cycle Speed Challenge; $2500 in bounties
===============================================

Cuckoo Cycle is the first graph-theoretic proof-of-work.

Proofs take the form of a length 42-cycle in a bipartite graph
with N nodes and N/2 edges, with N scalable from millions to billions and beyond.
This makes verification trivial: compute the 42x2 edge endpoints
with one initialising sha256 and 84 siphash24 computations, check that
each endpoint occurs twice, and that you come back to the
starting point only after traversing 42 edges.
A final sha256 can check whether the 42-cycle meets a difficulty target.

With siphash24 being a very lightweight hash function, this makes for
practically instant verification.

This is implemented in just 157 lines of C code (files cuckoo.h and cuckoo.c) at
https://github.com/tromp/cuckoo, where you can also find a whitepaper.

From this point of view, Cuckoo Cycle is a rather simple PoW.

Finding a 42-cycle, on the other hand, is far from trivial,
requiring considerable resources, and some luck
(for a given header, the odds of its graph having a 42-cycle are about 2.5%).

The algorithm implemented in cuckoo_miner.h runs in time linear in N.
(Note that running in sub-linear time is out of the question, as you could
only compute a fraction of all edges, and the odds of all 42 edges of a cycle
occurring in this fraction are astronomically small).
Memory-wise, it uses N/2 bits to maintain a subset of all edges (potential cycle edges)
and N additional bits (or N/2^k bits with corresponding slowdown)
to trim the subset in a series of edge trimming rounds.
Once the subset is small enough, an algorithm inspired by Cuckoo Hashing
is used to recognise all cycles, and recover those of the right length.

I'd like to claim that this implementation is a reasonably optimal Cuckoo miner,
and that trading off memory for running time, as implemented in tomato_miner.h,
incurs at least one order of magnitude extra slowdown.
I'd further like to claim that GPUs cannot achieve speed parity with server CPUs.

To that end, I offer the following bounties:

Speedup Bounty
--------------
$500 for an open source implementation that finds 42-cycles twice as fast (disregarding memory use).

Linear Time-Memory Trade-Off Bounty
-----------------------------------
$500 for an open source implementation that uses at most N/k bits while running up to 15 k times slower,
for any k>=2.

Both of these bounties require N ranging over {2^28,2^30,2^32} and #threads ranging over {1,2,4,8},
and further assume a high-end Intel Core i7 or Xeon and recent gcc compiler with regular flags as in my Makefile.

GPU Speed Parity Bounty
-----------------------
$1000 for an open source implementation for an AMD R9 280X or nVidia GeForce GTX 770 (or similar high-end GPU)
that is as fast as a high-end Intel Xeon running 16 threads. Again with N ranging over {2^28,2^30,2^32}.

Note that there is already a cuda_miner.cu, my attempted port of the edge trimming part
of cuckoo_miner.c to CUDA, but while it seems to run ok for medium graph sizes,
it crashes my computer at larger sizes, and I haven't had the chance to debug the issue yet
(I prefer to let my computer work on solving 8x8 connect-4:-)
Anyway, this could make a good starting point.

These bounties are to expire at the end of this year. They are admittedly modest in size, but then
claiming them might only require one or two insightful tweaks to my existing implementations.

I invite anyone who'd like to see my claims refuted to extend any of these bounties
with amounts of your crypto-currency of choice.

(I don't have any crypto-currencies to offer myself, but if need be, I might be able to convert
a bounty through a trusted 3rd party)

Happy bounty hunting!

-John
