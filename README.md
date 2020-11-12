# Cluster

This is the repository for the clusters of a Discluster system. It contains the raw TypeScript code for it, as well as outlining the protocol used for communication with parent CONTROL processes.

## Basic Mechanism

When a CONTROL server recieves instructions from MASTER to start spawning clusters, the cluster is created. The cluster will then recieve instructions to begin connecting its shards. After connecting, the cluster is responsible for the management of the shards, including event handling and shard resuming in case of disconnection.

### Initialisation

When the cluster is spawned, it will immediately spawn shards and connect them to the Discord gateway. CONTROL only spawns clusters when they are ready to connect to Discord. The cluster will connect shards at a rate of [1 per 5 seconds](https://discord.com/developers/docs/topics/gateway#identifying), unless [maximum concurrency](https://discord.com/developers/docs/topics/gateway#sharding-for-very-large-bots) is used in which case it will connect all shards at once.

After connecting all shards, the cluster will send an event back to CONTROL to signal that it is ready and recieving events.

### Operation

Cluster processes have two key roles: shard management and event handling.

The former involves making sure all shards are connected to the gateway. If a shard disconnects, it must be resumed (but this is usually handled by the shard itself). Some disconnect codes may be classed as 'fatal', a.k.a unrecoverable errors without user intervention. If this is the case, an event will be sent back to CONTROL for further processing. Refer to the CONTROL repository for more information.

The other key role is event handling. This involves receiving [dispatch packets](https://discord.com/developers/docs/topics/gateway#commands-and-events) from the Discord gateway and emitting them from the cluster itself, where the packet data can be used however the end user sees fit. By default, the cluster will take no further action on dispatch packets.