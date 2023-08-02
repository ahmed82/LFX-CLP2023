# Architecture
Making notes from the docs - https://hyperledger-fabric.readthedocs.io/en/release-2.5
## Basics of Hyperledger Fabric
* A private and permissioned blockchain, which has a ledger, uses smart contracts, and is a system by which participants manage their transactions.
* The members of a Hyperledger Fabric network enroll through a trusted Membership Service Provider (MSP).
* Ledger data can be stored in multiple formats, consensus mechanisms can be swapped in and out, and different MSPs are supported.
* Allows us to  create channels, allowing a group of participants to create a separate ledger of transactions. If two participants form a channel, then those participants — and no others — have copies of the ledger for that channel.
* Shared Ledger : Fabric has a ledger subsystem comprising two components: the world state and the transaction log. Each participant has a copy of the ledger to every Hyperledger Fabric network they belong to.  
  * World State : The database storing the state of the ledger at a given point in time. The data store is replaceable, the default being a LevelDB key-value store database
  * Transaction Log : Records all transactions which have resulted in the current value of the world state
  * The ledger is a combination of the world state database and the transaction log history.
* Smart Contracts :  
  * Fabric smart contracts are written in chaincode, invoked by an application external to the blockchain when that application needs to interact with the ledger
  * In most cases, chaincode interacts only with the database component of the ledger, the world state (querying it, for example), and not the transaction log.
  * Currently, Chaincode can be implemented in  Go, Node.js, and Java.
  * Privacy : Fabric supports networks where privacy (using channels) is a key operational requirement as well as networks that are comparatively open.
  * Consensus : Transactions must be written to the ledger in the order in which they occur, even though they might be between different sets of participants within the network. We need:
      1. The order of transactions must be established 
      2. A method for rejecting bad transactions that have been inserted into the ledger in error 
      * There are many ways to achieve it, each with different trade-offs 
      * For example, PBFT (Practical Byzantine Fault Tolerance) can provide a mechanism for file replicas to communicate with each other to keep each copy consistent, even in the event of corruption. Alternatively, in Bitcoin, ordering happens through a process called mining where competing computers race to solve a cryptographic puzzle which defines the order that all processes subsequently build upon.
      * Fabric has been designed to allow network starters to choose a consensus mechanism
## Fabric Model
The primary components of Fabric are:  
  1. Assets: Asset definitions enable the exchange of almost anything with monetary value over the network. 
     * Chaincode transactions can modify assets
     * Assets are represented in Hyperledger Fabric as a collection of key-value pairs, with state changes recorded as transactions on a Channel ledger. Assets can be represented in binary and/or JSON form.
  2. Chaincode : Chaincode is software defining an asset or assets and the transaction instructions for modifying the asset(s)
     * Chaincode functions execute against the ledger’s current state database and are initiated through a transaction proposal. 
     * Chaincode execution results in a set of key-value writes (write set) that can be submitted to the network and applied to the ledger on all peers.
     * Chaincode execution is partitioned from transaction ordering, limiting the required levels of trust and verification across node types, and optimizing network scalability and performance.
  3. Ledger Features : The ledger is the sequenced, tamper-resistant record of all state transitions in the fabric. State transitions are a result of chaincode invocations (‘transactions’) submitted by participating parties.Each transaction results in a set of asset key-value pairs that are committed to the ledger as creates, updates, or deletes.  

      The ledger is comprised of a blockchain (‘chain’) to store the immutable, sequenced record in blocks, as well as a state database to maintain current fabric state. There is one ledger per channel. Each peer maintains a copy of the ledger for each channel of which they are a member. 

      Features of the ledger:  
        1.  Query and update ledger using key-based lookups, range queries, and composite key queries  
        2.  Read-only queries using a rich query language (if using CouchDB as state database)
        3.  Read-only history queries — Query ledger history for a key, enabling data provenance scenarios
        4.  Transactions consist of the versions of keys/values that were read in chaincode (read set) and keys/values that were written in chaincode (write set)
        5.  Transactions contain signatures of every endorsing peer and are submitted to ordering service
        6.  Transactions are ordered into blocks and are “delivered” from an ordering service to peers on a channel
        7.  Peers validate transactions against endorsement policies and enforce the policies
        8.  Prior to appending a block, a versioning check is performed to ensure that states for assets that were read have not changed since chaincode execution time
        9.  There is immutability once a transaction is validated and committed
        10. A channel’s ledger contains a configuration block defining policies, access control lists, and other pertinent information
        11. Channels contain Membership Service Provider instances allowing for crypto materials to be derived from different certificate authorities
  4. Privacy : Fabric employs an immutable ledger on a per-channel basis, as well as chaincode that can manipulate and modify the current state of assets (i.e. update key-value pairs).  
      A ledger exists in the scope of a channel — 
      1. It can be shared across the entire network (assuming every participant is operating on one common channel) — or
      2. It can be privatized to include only a specific set of participants. In this case, these participants would create a separate channel and thereby isolate/segregate their transactions and ledger.  In order to solve scenarios that want to bridge the gap between total transparency and privacy, chaincode can be installed only on peers that need to access the asset states to perform reads and writes (in other words, if a chaincode is not installed on a peer, it will not be able to properly interface with the ledger).  

      When a subset of organizations on that channel need to keep their transaction data confidential, a private data collection (collection) is used to segregate this data in a private database, logically separate from the channel ledger, accessible only to the authorized subset of organizations.

      Thus, channels keep transactions private from the broader network whereas collections keep data private between subsets of organizations on the channel.

      To further obfuscate the data, values within chaincode can be encrypted (in part or in total) using common cryptographic algorithms such as AES before sending transactions to the ordering service and appending blocks to the ledger. Once encrypted data has been written to the ledger, it can be decrypted only by a user in possession of the corresponding key that was used to generate the cipher text.
  5. Security & Membership Services:  Fabric underpins a transactional network where all participants have known identities:
      *  Public Key Infrastructure is used to generate cryptographic certificates which are tied to organizations, network components, and end users or client applications.
      *  As a result, data access control can be manipulated and governed on the broader network and on channel levels. 
      *  This “permissioned” notion of Hyperledger Fabric, coupled with the existence and capabilities of channels, helps address scenarios where privacy and confidentiality are paramount concerns.
  6. Consensus : Consensus is achieved ultimately when the order and results of a block’s transactions have met the explicit policy criteria checks. https://hyperledger-fabric.readthedocs.io/en/release-2.5/fabric_model.html#consensus
  
    Now, I'll study the ordering service, since thats the part I need to understand for the mentorship project, will come back and cover the rest of the docs

 

