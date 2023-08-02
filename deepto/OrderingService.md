# The Ordering Service

## Ordering 
Many distributed blockchains, such as Ethereum and Bitcoin, are not permissioned, which means that any node can participate in the consensus process, wherein transactions are ordered and bundled into blocks. Because of this fact, these systems rely on probabilistic consensus algorithms which eventually guarantee ledger consistency to a high degree of probability, but which are still vulnerable to divergent ledgers (also known as a ledger “fork”), where different participants in the network have a different view of the accepted order of transactions.

Hyperledger Fabric works differently:
    * It features a node called an orderer (it’s also known as an “ordering node”) that does this transaction ordering, which along with other orderer nodes forms an ordering service. 
    * Because Fabric’s design relies on deterministic consensus algorithms, any block validated by the peer is guaranteed to be final and correct. Ledgers cannot fork the way they do in many other distributed and permissionless blockchain networks.
    *  Separating the endorsement of chaincode execution (which happens at the peers) from ordering gives Fabric advantages in performance and scalability, eliminating bottlenecks which can occur when execution and ordering are performed by the same nodes.

https://hyperledger-fabric.readthedocs.io/en/release-2.5/orderer/ordering_service.html#orderer-nodes-and-channel-configuration

## Orderers and the transaction flow

### Phase one: Transaction Proposal and Endorsement

