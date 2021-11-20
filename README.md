# Building exchange in java

exchange components (based of https://www.youtube.com/watch?v=b1e4t2k2KJY):
* primary me with orderbook per instrument (add/cancel/replace) - cancel - 40%
* message bus (in-memory but can be used aeron) - use multicast so once event happen (like exectuion) all component can receive it at the same time and return to clients
    * drop copy (of some specific clients)
    * MD (market data)
    * trade reporter (cause MD just show active orderbook, once order executed it just disappers from MD, so we need trade reporter)
* cancel manager (you can submit cancelRequest to cancel order at some point in future)
* auction manager (aggregate all prices & find the price that maximize the profit)
* secondary me (also called passive me) that listen output from primary me (this is important, it shouldn't listen incoming messages from message bus, only result of already processed messages by first me) and builds exactly the same state as first ME (so in case first me fail, we can start second me from exactly same place where first fail) => you can use paxos to decide who is primary me and when you should switch to secondary me. But no exchange use it, most exchanges has simple failover logic, if main me fail, promote passive me to primary.
* multi-threading - new thread per orderbook - good, but centralization has benefits
* state-machine recovery is a replay
* speed, latency, throughput, determenism - basic principle of exchange

### aeron (use it for as internal communication bus)
aeron + busy loop is faster then unix select (https://en.wikipedia.org/wiki/Select_(Unix))
if message size greater than MTU (1500byte) we need FragmentAssembler
aeron use `/dev/shm` => any file created there treated as simple file yet stored in-memory
since udp doens't gurantee order & delivery we aeron use position to ensure order & delivery
conductor on receiver side will check all position and if missing will send back NAK (negative acknolegement) and ask sender to re-send again
tcp philosophy - better late then never, low-latency philosophy - better never then late (so it's better not to send order at all, rather then send order with outdated price)
publisher write into log.buffer

### derivative exchange
This exchange will include following items:
* dated futures
* perpetual futures aka perpetual swap
* options
* auctions
* swaps (interest rate swap, like swap between fixed & floating income for defi or on perpetual 8 hour payment)
* move options
* spreads