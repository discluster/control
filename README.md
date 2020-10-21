# CONTROL

This is the repository for the CONTROL server for a Discluster system. It contains the raw TypeScript code for it, as well as outlining the protocol used for communication with individual cluster processes.

## Basic Mechanism

When the CONTROL server is started, it will begin attempting to connect to the MASTER server (as specified in configuration). If it fails to connect, it will retry every 30 seconds until a connection is established.
MASTER will send an acknowledgement packet back to the CONTROL processes to indicate that they should wait for further instructions.<br>
CONTROL processes will not perform any other actions until recieving information about the amount of shards the machine should serve.

### Initialisation

As soon as the amount of shards needed to be served by this machine is recieved, the CONTROL process will begin spawning cluster processes on the machine. The amount of cluster processes depends on how many shards were allocated, as well as if the number of shards per cluster is changed from the default of 10 to some other value due to [maximum concurrency](https://discord.com/developers/docs/topics/gateway#sharding-for-very-large-bots). Clusters will always serve the amount of shards as the maximum concurrency value if so.

If the allocated shards is not a multiple of shards per cluster, there will be one cluster that serves less shards than the rest of the clusters on that machine.

The CONTROL process is also responsible for initialising the connection of shards served by the clusters. CONTROL will not initialise the connection of shards in a cluster until indicated by MASTER that all clusters served on the machine should start. This is because Discord limits shard connections to [1 per 5 seconds](https://discord.com/developers/docs/topics/gateway#identifying). For bots that implement [maximum concurrency](https://discord.com/developers/docs/topics/gateway#sharding-for-very-large-bots), clusters will be able to connect all shards simultaneously instead of one at a time.<br>
Once all clusters are fully connected, the CONTROL process will send a packet back to MASTER indicating that it is operational.

### Operation

The CONTROL process continually monitors cluster processes to ensure that all shards are properly connected. If a cluster fails, it is the CONTROL process's responsibility to restart it.

If a cluster fails to properly start within three restarts, the CONTROL process will emit an event that can be handled however the end user sees fit. Please refer to the documentation (soon) for more information.