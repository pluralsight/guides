## Reborn 

### To build Reborn from scratch

1. Install Go on your machine [Go-getStarted](https://golang.org/doc/install)
2. Install & Start ZooKeeper
3. Get godep `go get github.com/tools/godep`
4. Get reborn `go get github.com/reborndb/reborn`
5. `cd` into reborn
6. Run command `make`
7. Run command `make gotest`
8. Run Command `make agent_test`
9. `cd` into /reborndb/reborn/sample 
10. Execute `startall.sh` script

### Customised Reborn Build for GAPI (To quickly start Reborn)

Here we have cloned Reborn from git repository and generated executable binary files using go.

1. Install Zookeeper `sudo apt-get install zookeeper`.
2. Clone the contents form RebornDB folder[RebornDB](https://github.dev.global.tesco.org/UKGrocery/Gapi_Redis/tree/master/RebornDB)
3. **`cd`** into **gapi_sample** folder.
4. Execute the below scripts in order
    -   `./start_zoo.sh`
    -   `./start_dashboard.sh`
    -   `./start_redis.sh`
    -   `./add_group.sh`
    -   `./initslot.sh`
    -   `./start_proxy.sh`
    -   `./set_proxy_online.sh`
5. open browser to http://localhost:18087/admin

>If you feel this is too much to configure -- just run one shell script `./startall.sh`

#### **Configuration**

** Configuration file : **
_reborn-proxy_, _reborn-config_ and _reborn-agent_ will take config.ini in current directory by default without a specific -c

```
config.ini

coordinator_addr=localhost:2181   <- Location of `zookeeper`, use `coordinator_addr=hostname1:2181,hostname2:2181,hostname3:2181,hostname4:2181,hostname5:2181` for `zookeeper` clusters. `coordinator_addr=http://hostname1:2181,http://hostname2:2181,http://hostname3:2181` for `etcd` clusters.
product=test                      <- Product name, also the name of this Coids clusters, can be considered as namespace, Reborn with different names have no intersection. 
dashboard_addr=localhost:18087    <- dashboard provides the RESTful API for CLI
coordinator=zookeeper             <-replace zookeeper to etcd if you are using etcd.

```

#### **Data Migration**

Reborn offers a reliable and transparent data migration mechanism, also it’s a killer feature which made Reborn distinguished from other static distributed Redis solution, such as Twemproxy.

The minimum data migration unit is key, we add some specific actions—such as SLOTSMGRT—to Reborn to support migration based on key, which will send a random record of a slot to another Reborn Server instance each time, after the transportation is confirmed the original record will be removed from slot and return slot’s length. The action is atomically.

For example: migrate data in slot with ID from 0 to 511 to server group 2, --delay is the sleep duration after each transportation of record, which is used to limit speed, default value is 0.

```
$ ../bin/reborn-config slot migrate 0 511 2 --delay=10
```
Migration progress is reliable and transparent, data won’t vanish and top layer application won’t terminate service.

#### **Auto Rebalance**

Reborn support dynamic slots migration based on RAM usage to balance data distribution.
```
$../bin/reborn-config slot rebalance
```
Requirements:

+  All reborn-server must set maxmemory.
+ All slots’ status should be online, namely no transportation task is running.
+ All server groups must have a master.

#### ** Failover **

reborn-agent is a monitoring and HA tool for Reborn. By using ZooKeeper, reborn-agent elects a leader to achieve high high availability. reborn-agent will do failover when either reborn-server slave or reborn-server master node is down.
```
$ ../bin/reborn-agent -c config.ini -L ./log/agent.log --http-addr=0.0.0.0:39000 --ha
```

