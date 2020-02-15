## Elasticsearch Installation and Configuration for Production
This role installs elasticsearch on CentOS/Debian based servers.

### How to use?

To install elasticsearch clone the role:
```
git clone git@github.com:sinahamedheidari/elasticsearch-ansible.git
```

Modify hosts and play the role:
```
ansible-play play.yml
```

## System Configurations
### Disable swapping
Swapping is very bad for performance, for node stability, and should be avoided at all costs. It can cause garbage collections to last for minutes instead of milliseconds and can cause nodes to respond slowly or even to disconnect from the cluster.
```
swapoff -a
```
### Virtual memory
Elasticsearch uses a mmapfs directory by default to store its indices. The default operating system limits on ```mmap``` counts is likely to be too low, which may result in out of memory exceptions.
```
sysctl -w vm.max_map_count=262144
```
### File descriptors
set ```ulimit -n 65535``` as root before starting Elasticsearch, or set ```nofile to 65535``` in ```/etc/security/limits.conf```

### Number of threads
This can be done by setting ```ulimit -u 4096``` as root before starting Elasticsearch, or by setting ```nproc to 4096``` in ```/etc/security/limits.conf```

## Elasticsearch Configurations

### path.data and path.logs
In production use, you will almost certainly want to change the locations of the data and log folder:
```
path:
   logs: /var/log/elasticsearch
   data: /var/data/elasticsearch
```

The path.data settings can be set to multiple paths, in which case all paths will be used to store data (although the files belonging to a single shard will all be stored on the same data path):
```
path:
  data:
    - /mnt/elasticsearch_1
    - /mnt/elasticsearch_2
    - /mnt/elasticsearch_3
```
### cluster.name
A node can only join a cluster when it shares its cluster.name with all the other nodes in the cluster. 
```
cluster.name: <your_cluster_name>
```
### node.name
Elasticsearch uses node.name as a human readable identifier for a particular instance of Elasticsearch so it is included in the response of many APIs. It defaults to the hostname that the machine has when Elasticsearch starts.
```
node.name: prod-data-2
```
### network.host
In production in order to form a cluster with nodes on other servers your node will need to bind to a non-loopback address.
```
network.host: 192.168.1.10
```
### Discovery and cluster information settings
When you want to form a cluster with nodes on other hosts, you must use the ```discovery.seed_hosts``` setting to provide a list of other nodes in the cluster that are master-eligible and likely to be live.

In production mode, you must explicitly list the master-eligible nodes whose votes should be counted in the very first election. This list is set using the ```cluster.initial_master_nodes``` setting.
#### Note: You should not use this setting when restarting a cluster or adding a new node to an existing cluster.
If the port is not specified, elasticsearch will scan ports 9300 to 9305 to try to connect to other nodes on the cluster.
```
discovery.seed_hosts:
   - 192.168.1.10:9300
   - 192.168.1.11 
   - seeds.mydomain.com 
cluster.initial_master_nodes: 
   - master-node-a
   - master-node-b
   - master-node-c
```
### Setting the heap size
By default, Elasticsearch tells the JVM to use a heap with a minimum and maximum size of 1 GB. Elasticsearch will assign the entire heap specified in ```jvm.options``` via the Xms (minimum heap size) and Xmx (maximum heap size) settings.

#### Hint: Set Xmx and Xms to no more than 50% of your physical RAM. Elasticsearch requires memory for purposes other than the JVM heap and it is important to leave space for this.

set the heap size via the jvm.options file:
```
-Xms2g 
-Xmx2g 
```

### Start Elasticsearch
```
systemctl start elasticsearch.service
systemctl enable elasticsearch.service
```
### Stop Elasticsearch
```
systemctl stop elasticsearch.service
```

### Adding nodes to a cluster


1. Set up a new Elasticsearch instance.
2. Specify the name of the cluster in its ```cluster.name``` attribute in ```elasticsearch.yml```.
3. Start Elasticsearch. The node automatically discovers and joins the specified cluster.

### Full cluster restart and rolling restart

Check [here](https://www.elastic.co/guide/en/elasticsearch/reference/current/restart-cluster.html)