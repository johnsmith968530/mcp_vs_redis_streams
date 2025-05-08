* `$Source: /home/x/Dropbox/3/Documents/MCP_vs_Redis_Streams/RCS/README.md,v $`
* `$Date: 2025/05/08 06:25:18 $`
* `$Revision: 1.2 $`

# MCP vs Message Bus Architecture w/ Redis Streams

## Why?
* MCP is mostly 1 client process controlled by the user and multiple server processes communicating via stdout. Interactions initiated by the client. Usually runs on a single machine, but protocol has simple networking provisions.
  * Painful to debug because you don't know if the problem is with the MCP client or the MCP server.
    * Hard to write unit tests for the MCP server.
  * Inefficient when you have many concurrent users, as most MCP server processes will sit idle.
  * No automated retries, no automated failover, no concurrency.
  * What if you have an AI agent as an MCP server, and that agent wants to call other AI agents as MCP servers? You'll end up with a whole bunch of unnecessary processes.
* Typical real-world solution is to use something like a message queue.
  * Create a bridge between MCP and the queue.
* I prefer Redis Streams
  * Has a push mechanism so the servers don't have to constantly poll, which minimizes latency.
  * Has worker pools, so if a worker process crashes, another worker (perhaps on a different machine) will retry the task.
  * Using Redis Streams is nontrivial, since there are multiple race conditions that you have to be wary of.
    * Beyond the scope of this talk!
* Other gotchas
  * Rust is statically typed, so if your messages are JSON, Rust doesn't have nested dictionary types like Python or Node.
    * I wound up keeping the JSON string as a string, and deserializing the portions I needed on an as-needed basis. Ugh!
  * Redis has a key-value store type, but they implemented it on top of a 1-D array, so it also doesn't have the nested dictionary and nested arrays like Python and Node.
    * When I tried serializing and de-serializing a nested dictionary in Rust + Redis, I got back a dictionary of JSON strings. Ugh!
    * I wound up keeping the JSON as a single string value in Redis.
  * I'd recommend using the same version of the MCP libraries across all the clients and servers in the system, as the code seems to be evolving rapidly. I ran into a situation where some AI-generated code pulled in an older version that was subtly incompatible with newer versions and it took forever to figure out what was going on.

* vim: set et ff=unix ft=markdown nocp sts=2 sw=2 ts=2:
