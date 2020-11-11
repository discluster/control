# CONTROL

This is the repository for the CONTROL server for a Discluster system. It contains the raw TypeScript code for it, as well as outlining the protocol used for communication with individual cluster processes.

## Basic Mechanism

When the CONTROL server is started, it will begin attempting to connect to the MASTER server (as specified in configuration). If it fails to connect, it will retry every 30 seconds until a connection is established.
MASTER will send an acknowledgement packet back to the CONTROL processes to indicate that they should wait for further instructions.<br>
CONTROL processes will not perform any other actions until receiving information about the amount of shards the machine should serve.

### Initialisation
The CONTROL server does nothing of use before it recieves a specific packet from MASTER instructing it to connect to the Discord gateway. The only things that happen are that it connects to MASTER and recieves some information from MASTER about how to connect to Discord. (refer to https://github.com/discluster/master/blob/main/WEBSOCKET_COMMUNICATION.md for details)

As soon as the amount of shards needed to be served by this machine is received, the CONTROL process will begin spawning cluster processes on the machine using the token provided by MASTER. The amount of cluster processes depends on how many shards were allocated, as well as if the number of shards per cluster is changed from the default of 10 to some other value due to [maximum concurrency](https://discord.com/developers/docs/topics/gateway#sharding-for-very-large-bots). Clusters will always serve the amount of shards as the maximum concurrency value if so.

If the allocated shards is not a multiple of shards per cluster, there will be one cluster that serves less shards than the rest of the clusters on that machine.

The CONTROL process is also responsible for initialising the connection of shards served by the clusters. This happens as soon as a cluster is fully spawned. Clusters are only spawned and connected to Discord when explicitly signalled to do so by MASTER. This is because Discord limits shard connections to [1 per 5 seconds](https://discord.com/developers/docs/topics/gateway#identifying). For bots that implement [maximum concurrency](https://discord.com/developers/docs/topics/gateway#sharding-for-very-large-bots), clusters will be able to connect all shards simultaneously instead of one at a time.<br>
Once all clusters are fully connected, the CONTROL process will send a packet back to MASTER indicating that it is operational.

### Operation

The CONTROL process continually monitors cluster processes to ensure that all shards are properly connected. If a cluster fails, it is the CONTROL process's responsibility to restart it.<br>
Clusters may also pass error events back up to CONTROL in case of unrecoverable problems. One such example is if an invalid token (or intent) is provided. All such errors are passed to MASTER, which will usually result in MASTER sending instructions to the entire system to stop execution, since such errors will usually affect all shards instead of just one or two.

If a cluster fails to properly start within three restarts, the CONTROL process will emit an event that can be handled however the end user sees fit. Please refer to the documentation (soon) for more information.
