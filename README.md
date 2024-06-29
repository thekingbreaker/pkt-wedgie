# PKT Blockchain Audit Report (pkt-wedgie)
## Hardnonce Bypass / Fake Announcement Production
audit performed by TheKingBreaker (github.com/thekingbreaker)

A concerning flaw has been identified in the PKT algorithm with regards to production/validation of announcement
mining. The announcement mining relies on two levels of work (hard/soft). This flaw manages to skip the hard work
part. It allows a miner to produce higher volume as well as fake announcements, effectively breaking the PKT
blockchain consensus. In theory, should be fairly trivial for a cloud miner/carder to create their own private pool to take
advantage of the higher volume of fake announcements. They can internal mine using a very low work target. It is also
possible to create fake announcements at extremely high work targets, but the volume will be smaller.

Please refer to diagram below which shows how normal operation works using the standard packetcrypt-rs
implementation. In order to search for valid announcements, the miner needs to perform hard round + soft round per thread.

![Alt text](diag_1.png?raw=true "diagram of announcement mining process")

The hard round involves building a merkle tree containing 8192 randhash items.  
This is performed for every new soft round, adding a throttle to the amount of announcements a miner can produce.  
The issue with this design is that the block miner blindly accepts whatever merkle root hash that the announcement miner provides.
The announcement miner can fake the 8192 items.  In this demonstration we simply modify one item every round in order to force a new merkle root hash.
This means the previously computed randhash items can be recycled (free) - only need to rebuild the merkle tree.

In the diagram below we can see how mining algorithm can be abused to create high volume of fake
announcements. With this setup all threads only need a single hard round. The hard nonce value is
no longer used - it can be set to whatever value the miner wants.

![Alt text](diag_2.png?raw=true "diagram of announcement mining process using mentioned flaw")

This report comes along with sourcecode (src.7z release) that demonstrates this flaw in a live environment. The
sourcecode is a modified version of packetcrypt-rs, it has been tweaked to make it easier to see how the flaw can be
implemented.

To build:
```bash
CC=clang cargo –release –features jemalloc –features jit
```

To run
```bash
./target/release/packetcrypt ann http://pool.pkt.world/2048 –paymentaddr <your wallet here>
```

At a worktarget of 2048 it should start creating fake announcements fairly quickly.
Once its up and running, wait a bit and you should see something like screenshot below:

![Alt text](console_1.png?raw=true "example output of uploading fake annnouncements to pool")

