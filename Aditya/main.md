# Integration Work

Currently, I have the `bdls-fabric` repository containing the `etcdraft` and `smartbft`. I will start with importing the `bdls` repository in this respository.

## Importing BDLS

```bash
$ go get github.com/BDLS-bft/bdls
$ go mod tidy
$ go mod vendor
```

## Fabric Source Code Study Notes:-

[Helpful article](https://www.cnblogs.com/CherryTab/p/13796254.html)

- **Ledger**: In order to tolerate crash faults, orderer uses file-based ledger to persist blocks on the file system. The block locations on disk are 'indexed' in a lightweight LevelDB database by number so that clients can efficiently retrieve a block by number.

### Registrar

Registrar serves as a point of access and control for the individual channel resources. It contains mainly two resources: `chains` and `consenters`.

```go
// Registrar serves as a point of access and control for the individual channel resources.
type Registrar struct {
	config localconfig.TopLevel

	lock      sync.RWMutex
	chains    map[string]*ChainSupport
	followers map[string]*follower.Chain
	// existence indicates removal is in-progress or failed
	// when failed, the status will indicate failed all other states
	// denote an in-progress removal
	pendingRemoval map[string]consensus.StaticStatusReporter

	consenters                  map[string]consensus.Consenter
	ledgerFactory               blockledger.Factory
	signer                      identity.SignerSerializer
	blockcutterMetrics          *blockcutter.Metrics
	callbacks                   []channelconfig.BundleActor
	bccsp                       bccsp.BCCSP
	clusterDialer               *cluster.PredicateDialer
	channelParticipationMetrics *Metrics

	joinBlockFileRepo *filerepo.Repo
}
```
### ChainSupport
- `ChainSupport` holds the resources for a particular channel. ChainSupport brings together all the resources needed by a channel, so a ChainSupport represents a chain.

  ```go
  type ChainSupport struct {
  *ledgerResources
  msgprocessor.Processor
  *BlockWriter
  consensus.Chain
  cutter blockcutter.Receiver
  identity.SignerSerializer
  BCCSP bccsp.BCCSP

      // NOTE: It makes sense to add this to the ChainSupport since the design of Registrar does not assume
      // that there is a single consensus type at this orderer node and therefore the resolution of
      // the consensus type too happens only at the ChainSupport level.
      consensus.MetadataValidator

      // The registrar is not aware of the exact type that the Chain is, e.g. etcdraft, inactive, or follower.
      // Therefore, we let each chain report its cluster relation and status through this interface. Non cluster
      // type chains (solo) are assigned a static reporter.
      consensus.StatusReporter

  }
  ```
#### Chain
- **Chain** is an interface. It's implementation is not a chain, but a consensus instance of a chain. *It can be Solo, Kafka, and EtcdRaft*. It runs in a separate coroutine, uses Channel and ChainSupport to communicate, and calls other interfaces to complete block cutting. And let all Orderer nodes agree on the transaction.

```go
type Chain interface {
	Order(env *cb.Envelope, configSeq uint64) error

	Configure(config *cb.Envelope, configSeq uint64) error

	WaitReady() error

	Errored() <-chan struct{}

	Start()

	Halt()
}

```

### Consenter
Consenter is also an interface, and it has only one function to create a Chain. **Each consensus plug-in has its own separate consenter implementation, which is used to create solo instances, kafka instances or etcdraft instances respectively**.
```go
// Consenter defines the backing ordering mechanism.
type Consenter interface {
	HandleChain(support ConsenterSupport, metadata *cb.Metadata) (Chain, error)
}
```

### ConsenterSupport
ConsenterSupport provides the resources available to a Consenter implementation.

```go
type ConsenterSupport interface {
	identity.SignerSerializer
	msgprocessor.Processor

	// SignatureVerifier verifies a signature of a block.
	SignatureVerifier() protoutil.BlockVerifierFunc

	// BlockCutter returns the block cutting helper for this channel.
	BlockCutter() blockcutter.Receiver

	// SharedConfig provides the shared config from the channel's current config block.
	SharedConfig() channelconfig.Orderer

	// ChannelConfig provides the channel config from the channel's current config block.
	ChannelConfig() channelconfig.Channel

	// CreateNextBlock takes a list of messages and creates the next block based on the block with highest block number committed to the ledger
	// Note that either WriteBlock or WriteConfigBlock must be called before invoking this method a second time.
	CreateNextBlock(messages []*cb.Envelope) *cb.Block

	// Block returns a block with the given number,
	// or nil if such a block doesn't exist.
	Block(number uint64) *cb.Block

	// WriteBlock commits a block to the ledger.
	WriteBlock(block *cb.Block, encodedMetadataValue []byte)

	// WriteConfigBlock commits a block to the ledger, and applies the config update inside.
	WriteConfigBlock(block *cb.Block, encodedMetadataValue []byte)

	// Sequence returns the current config sequence.
	Sequence() uint64

	// ChannelID returns the channel ID this support is associated with.
	ChannelID() string

	// Height returns the number of blocks in the chain this channel is associated with.
	Height() uint64

	// Append appends a new block to the ledger in its raw form,
	// unlike WriteBlock that also mutates its metadata.
	Append(block *cb.Block) error
}
```

### Summary of Registrar: 
- Registrar is all-inclusive, mainly `ChainSupport` and `Consenter`, Consenter is pluggable
- `ChainSupport` represents a chain and can point to a consensus instance(`consensus.Chain`) belonging to this chain, which is created by a `Consenter` of the corresponding consensus type.
- The consensus instance uses `ConsenterSupport` to access consensus external resources.


### Orderer Startup Process
The steps in the startup process of the orderer are:
- load configuration file: Load parses the orderer YAML file and environment, producing a struct suitable for config use, returning error on failure.
- Set Logger
- Set up a local MSP `(msp for the orderer nodes)`
- The core startup part:
	- load the genesis block
	- Create ledger factory
	- Create a native gRPCServer
	- If the consensus requires a cluster (raft), create a cluster gRPCServer
	- Create Registrar: Set up the consensus plug-in, start each channel, and if the consensus is raft, also set the - cluster's gRPC interface processing function Step
	- Create a local server: it is the processing service of atomic broadcast, which integrates Broadcast processing - function, deliver processing function and registrar
	- open profile
	- Start the cluster gRPC service
	- Start the native gRPC service



#### Start the entry main
github.com\hyperledger\fabric\orderer\common\server\main.go

```go
// Main is the entry point of orderer process
func Main() {
	fullCmd := kingpin.MustParse(app.Parse(os.Args[1:]))

	// "version" command
	if fullCmd == version.FullCommand() {
		fmt.Println(metadata.GetVersionInfo())
		return
	}

	// loading local config
	conf, err := localconfig.Load()
	if err != nil {
		logger.Error("failed to parse config: ", err)
		os.Exit(1)
	}
	// initializing logging
	initializeLogging()
	// 
	initializeLocalMsp(conf)

	prettyPrintStruct(conf)
	// a function which is written inline in modern fabric version
	Start(fullCmd, conf)
}

```

## Miscelleneous

> ***BCCSP*** is the blockchain cryptographic service provider that offers
 the implementation of cryptographic standards and algorithms.


### Write Ahead Logs in Fabric: 
In the context of Hyperledger Fabric, a Write-Ahead Log (WAL) is a crucial component of the blockchain framework that helps ensure data consistency and durability. The WAL mechanism is used to maintain a sequential record of transactions before they are committed to the actual blockchain ledger.

Here's how the Write-Ahead Log works in Hyperledger Fabric:

1. **Transaction Proposal and Ordering:** In Hyperledger Fabric, before a transaction is added to the ledger, it goes through the process of endorsement and ordering. During endorsement, the transaction is executed and verified by endorsing peers. After endorsement, the transactions are ordered into a block by the ordering service.

2. **Write-Ahead Log Creation:** Once the transactions are ordered into a block by the ordering service, they are temporarily stored in the Write-Ahead Log. The Write-Ahead Log is a sequential record of the transactions in the order they were received.

3. **Validation and Commitment:** After transactions are recorded in the Write-Ahead Log, they are sent to the validating peers. The validating peers validate the transactions, ensuring that they meet the endorsement policies and are consistent with the current state of the ledger.

4. **Commitment to the Ledger:** Once transactions are successfully validated, they are considered valid and are committed to the actual blockchain ledger. This means the transactions become a permanent part of the ledger's history and cannot be altered.

5. **Checkpointing:** Periodically, the Write-Ahead Log is "checkpointed." Checkpointing involves saving a snapshot of the current state of the ledger and clearing the log of already committed transactions. This process helps manage the growth of the Write-Ahead Log and maintain efficient performance.

The Write-Ahead Log mechanism in Hyperledger Fabric ensures that transactions are recorded in a sequential and durable manner. If a crash or failure occurs, the system can recover by replaying the transactions from the Write-Ahead Log to bring the ledger back to a consistent state.

The Write-Ahead Log plays a critical role in providing fault tolerance, data consistency, and recovery mechanisms in Hyperledger Fabric, making it an integral part of the framework's architecture.


