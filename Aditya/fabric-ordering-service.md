# Fabric Ordering Service

- Fabric design relies on deterministic consensus algorithms, any block validated by the peer is guaranteed to be final and correct. 
- Orderers, in addition to the ordering role, has a list of organizations - `consortium`, who can create a channel.

In the whole transaction flow, the fabric ordering service comes in phase two - `Ordering and packaging transactions into blocks`


1. After the completion of the first phase of a transaction, a client application has received an endorsed transaction proposal response from a set of peers. It’s now time for the second phase of a transaction.

2. In this phase, application clients submit transactions containing endorsed transaction proposal responses to an ordering service node. 

3. The ordering service creates blocks of transactions which will ultimately be distributed to all peers on the channel for final validation and commit in phase three.

4. Ordering service nodes receive `transactions from many different application clients concurrently`. These ordering service nodes work together to collectively form the ordering service. Its job is to arrange batches of submitted transactions into a well-defined sequence and package them into blocks. These blocks will become the blocks of the blockchain!


5. The number of transactions in a block depends on channel configuration parameters related to the desired size and maximum elapsed duration for a block (`BatchSize` and `BatchTimeout` parameters, to be exact). 

6. The blocks are then saved to the orderer’s ledger and distributed to all peers that have joined the channel.

Two work of orderer: 
- collect, cut and order and pack the transactions into blocks, and 
- distribute the blocks to peers


## Ordering Service Implementation
While every ordering service currently available handles transactions and configuration updates the same way, there are nevertheless several different implementations for achieving consensus on the strict ordering of transactions between ordering service nodes.

> Standing up an Ordering Node: https://hyperledger-fabric.readthedocs.io/en/release-2.2/orderer_deploy.html

### Terms in Raft: 
- Log Entry - leader injects the logs and followers have to replicate them
- Consenter set - set of orderer nodes which are part of raft consensus
- Finite-State Machine - for the order and validity of logs
- Quorum - 50 % (majority votes for the order of transaction)
- Leader - It's a role and not a special orderer node
- Follower - They replicate the logs sent by the leader nodes

### Raft in a transaction Flow
- Every channel runs on a separate instance of Raft protocol, which allows each instance to elect a different leader. 

-  This configuration also allows further decentralization of the service in use cases where clusters are made up of ordering nodes controlled by different organizations. 

- While all Raft nodes must be part of the system channel, they do not necessarily have to be part of all application channels.

- Channel creators (and channel admins) have the ability to pick a subset of the available orderers and to add or remove ordering nodes as needed (as long as only a single node is added or removed at a time).

- While this configuration creates more overhead in the form of redundant heartbeat messages and goroutines, it lays necessary groundwork for BFT.

- In Raft, transactions (in the form of proposals or configuration updates) are automatically routed by the `ordering node that receives the transaction to the current leader of that channel`. This means that `peers and applications do not need to know who the leader node is at any particular time`. Only the ordering nodes need to know.

- When the orderer validation checks have been completed, the transactions are ordered, packaged into blocks, consented on, and distributed, as described in phase two of our transaction flow.

## Architectural Notes
### Leader Election in Raft
https://hyperledger-fabric.readthedocs.io/en/release-2.2/orderer/ordering_service.html#how-leader-election-works-in-raft

http://thesecretlivesofdata.com/raft/


### Snapshots
They helps when an ordering node goes down and miss the logs when it is restarted.

- While it’s possible to keep all logs indefinitely, in order to save disk space, Raft uses a process called “snapshotting”, in which users can define how many bytes of data will be kept in the log. This amount of data will conform to a certain number of blocks (which depends on the amount of data in the blocks. Note that only full blocks are stored in a snapshot).

- For example, let’s say lagging replica R1 was just reconnected to the network. Its latest block is 100. Leader L is at block 196, and is configured to snapshot at amount of data that in this case represents 20 blocks. R1 would therefore receive block 180 from L and then make a Deliver request for blocks 101 to 180. Blocks 180 to 196 would then be replicated to R1 through the normal Raft protocol.

