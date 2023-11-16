# copious-session-cluster

An extension of global_sessions providing cluster management, decay rings and quota buffering.

## Purpose

Using configuration and some basic principles (features) the aim of this package is to provide an easily used CLI tool for setting up a shared memory hash table in a cluster with two kinds of access. The access types might be thought of as shards or as blackboards. Let's list these:

* single CPU running multiple processes on cores  - shared memory
* a number of CPUs with networking and replication - shards

The running processes will manage one or more shared memory regions used as hash tables. It should be easy to bring up the main managing processes governing the shared memory regions. It should be easy to provide a list of IPs and PORTS within a cluster and bring up a sharded version of the hash tables. Proximity of a process to a cluster, where the process has a a particular function, should help determine the degree to which a table is responsive to queries old and new. 

A startup and configuration template is provided. Deference to **/etc** file entries for startup with system startup should be provided.

>Def: LRU means least recently used. (not to forget)

## Extended LRU

The ***session cluster*** can focus on a single sized object and treat all shards of the table as a single key-value map. So, in this documentation we may refer to the **session object** as the type of object stored. This is a fixed sized object. This module can handler other functional types of objects with fixed size. Another module can deal with varying sized objects.

**Session objects**, as well as other types of objects, can be placed in LRUs, which allow them to reside in memory as long as processes have use for them. The aim of the LRU operation is to keep memory constant for operations while the load of outside influences varies. An LRU may fill up as new data comes in. So, the LRU might be extended to allow session storage to increase temporarily or to make use of secondary reserved storage or high preasure situations. 

This module operates with rules for adding and removing shards and for displacing session objects to secondary shards and bringing them back in.

### TTL Rings

All session objects will have a lifetime. They begin their lives close to the client. But, as the client loses interest in them, their time to live will decay. If the client is too interested in them, they may be placed in holding queues set aside with the purpose of slowing down some applications.

Depending on the rate of access, the TTL will decay. This module may be configured to move sessions to shards assigned to tiers of decay. The secondary, tertiary, etc. shards may be placed further and further from the client associated with them. We may think of these shards as TTL rings with some notion of expanding diameter. 

If a session object is requested after it has been moved into an outer ring, the module may be configured to move the session object closer or into the primary ring, the share memory section accessble from the client facing server.

### Quotas and Buffering

The smallest diameter (**SD**) TTL Ring may contain session objects that are being requested at a rate determined to be higher than desirable for successful service sharing and load balancing.

These objects may be removed from the **SD** TTL Ring and placed into a suspect storage. The suspect storage can operate in the reverse of a decaying TTL rings moving suspect session objects into the access of a process that shuts down sessions. Processes separate from the client facing process can manage the rapidly firing sessions. These processes can monitor session objects that may move back into the SD TTL Ring LRU if they cool down.

### Process Sharing

More than on process is possible for client facing services. At least one process (and not many) in the cluster will be assigned the duty to start and stop sessions when clients request and pass authorization. Other processes may handle tokenized operations on behalf of the authorization and accesss the SD TTL Ring or request a session lookup. These clients do not have to be exposed to the negotiation required for accessing hot and cold sessions, but may expect their library interface to manage the interactions.

Each CPU separated by networking should be initialized with a single global-session process. The process may run as a secondary service managing sections designated to larger diameter tiers. But if a client facing processes starts up on the same CPU, the global-session process should ensure the allocation of the SD TTL Ring memory section and operate as a manager of low latency responses to session objects.



