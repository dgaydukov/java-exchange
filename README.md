# Building exchange in java

exchange componets (based of https://www.youtube.com/watch?v=b1e4t2k2KJY):
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