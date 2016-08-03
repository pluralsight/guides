### Prerequisites

1. Install Redis Version greater that 2.9.

### Starting Dynomite Cluster

1. Modify ** StartServers.sh ** and **StopServers.sh** files under **Redis\_Instances** folder. Change the `filepath` variable value to `$PWD/Redis_instances/redis_$dirname`.
2. Assign _execute_ permission to shell scripts.
3. Execute _StartServers.sh_ to start 6 Redis servers as Masters on port 3001 to 3006.
4. To verfiy run `ps -eaf | grep redis` command
4. Modify **StartDynomiteCluster.sh** , change `filepath` variable value to Current working directory, and assign _execute_ permission.
5. Execute **StartDynomiteCluster.sh** to start Dynomite cluster with 2 Racks, each having 3 Dynomite nodes.


### REST API's

+  Check Dynomite node info running on stats-Port 22221. **localhost:22221/info**
+  Check Dynomite cluster details **localhost:22221/cluster_describe**


### Dynomite Configuration terminologies

Dynomite can be configured through a YAML file specified by the -c or --conf-file command-line argument on process start. The configuration files parses and understands the following keys:

+ **env**: Specify environment of a node.  Currently supports aws and network (for physical datacenter).
+ **datacenter**: The name of the datacenter.  Please refer to [architecture document](https://github.com/Netflix/dynomite/wiki/Architecture).
+ **rack**: The name of the rack.  Please refer to [architecture document](https://github.com/Netflix/dynomite/wiki/Architecture).
+ **dyn_listen**: The port that dynomite nodes use to inter-communicate and gossip.
+ **gos_interval**: The sleeping time in milliseconds at the end of a gossip round.
+ **tokens**: The token(s) owned by a node.  Currently, we don't support vnode yet so this only works with one token for the time being.
+ **dyn_seed_provider**: A seed provider implementation to provide a list of seed nodes.
+ **dyn_seeds**: A list of seed nodes in the format: address:port:rack:dc:tokens (node that vnode is not supported yet)
+ **listen**: The listening address and port (name:port or ip:port) for this server pool.
+ **timeout**: The timeout value in msec that we wait for to establish a connection to the server or receive a response from a server. By default, we wait indefinitely.
+ **preconnect**: A boolean value that controls if dynomite should preconnect to all the servers in this pool on process start. Defaults to false.
+ **data_store**: An integer value that controls if a server pool speaks redis (0) or memcached (1) or other protocol. Defaults to redis (0).
+ **server_connections**: The maximum number of connections that can be opened to each server. By default, we open at most 1 server connection.
+ **auto_eject_hosts**: A boolean value that controls if server should be ejected temporarily when it fails consecutively server_failure_limit times.for information. Defaults to false.
+ **server_retry_timeout**: The timeout value in msec to wait for before retrying on a temporarily ejected server, when auto_eject_host is set to true. Defaults to 30000 msec.
+ **server_failure_limit**: The number of consecutive failures on a server that would lead to it being temporarily ejected when auto_eject_host is set to true. Defaults to 2.
+ **servers**: A list of local server address, port and weight (name:port:weight or ip:port:weight) for this server pool. Usually there is just one.
+ **secure_server_option**: Encrypted communication. Must be one of 'none', 'rack', 'datacenter', or 'all'.
> For more documentation on Dynomite and its configuration visit Github wiki page [Dynomite/Wiki](https://github.com/Netflix/dynomite/wiki)