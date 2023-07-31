# Exploring Solo Orderer

- We have a `logger` for logging information - created from `flogging.MustGetLogger`.
- We have an empty consenter struct (possibly because this is `solo` - one ordering node for each channel only)

#### Chain Struct
```go
type chain struct {
    // ConsenterSupport provides the resources available to a Consenter implementation.
	support  consensus.ConsenterSupport 
	sendChan chan *message
	exitChan chan struct{}
}
```

#### Message Struct 
```go
type message struct {
	configSeq uint64
	normalMsg *cb.Envelope // normal trasactions (has payload, signatures, endorsments, etc)
	configMsg *cb.Envelope // transacitons which update the configuration. (has similar things)
}
```

Then we have a `HandleChain()` function which takes the `ConsenterSupport` and `metadata` and returns a new chain initialized with this information. 

### The chain interface: 
```go

// Chain defines a way to inject messages for ordering.
// Note, that in order to allow flexibility in the implementation, it is the responsibility of the implementer
// to take the ordered messages, send them through the blockcutter.Receiver supplied via HandleChain to cut blocks,
// and ultimately write the ledger also supplied via HandleChain.  This design allows for two primary flows
// 1. Messages are ordered into a stream, the stream is cut into blocks, the blocks are committed (solo, kafka)
// 2. Messages are cut into blocks, the blocks are ordered, then the blocks are committed (sbft)
type Chain interface {
	// Order accepts a message which has been processed at a given configSeq.
	// If the configSeq advances, it is the responsibility of the consenter
	// to revalidate and potentially discard the message
	// The consenter may return an error, indicating the message was not accepted
	Order(env *cb.Envelope, configSeq uint64) error

	// Configure accepts a message which reconfigures the channel and will
	// trigger an update to the configSeq if committed.  The configuration must have
	// been triggered by a ConfigUpdate message. If the config sequence advances,
	// it is the responsibility of the consenter to recompute the resulting config,
	// discarding the message if the reconfiguration is no longer valid.
	// The consenter may return an error, indicating the message was not accepted
	Configure(config *cb.Envelope, configSeq uint64) error

	// WaitReady blocks waiting for consenter to be ready for accepting new messages.
	// This is useful when consenter needs to temporarily block ingress messages so
	// that in-flight messages can be consumed. It could return error if consenter is
	// in erroneous states. If this blocking behavior is not desired, consenter could
	// simply return nil.
	WaitReady() error

	// Errored returns a channel which will close when an error has occurred.
	// This is especially useful for the Deliver client, who must terminate waiting
	// clients when the consenter is not up to date.
	Errored() <-chan struct{}

	// Start should allocate whatever resources are needed for staying up to date with the chain.
	// Typically, this involves creating a thread which reads from the ordering source, passes those
	// messages to a block cutter, and writes the resulting blocks to the ledger.
	Start()

	// Halt frees the resources which were allocated for this Chain.
	Halt()
}

```
#### Functions in the Chain Interface: 

1. **Order**: The `configSeq` is the identifier to recognize a particular configuration. If the configuration is changed then any message received with previous configuration is discarded. This is the responsibility of the orderer. This check happens in the `Order` function.

2. **Configure**: For updating the configuration of a particular channel.

3. **WaitReady**: Wait for consenter to be ready, until it processes in-flight messages. Return nil if this functionality is not required. 

4. **Errored**: Returns a channel, which can be closed when an error has occurred. 

5. **Start**: It should allocate whatever resources are needed for staying up to date with the chain. 
- Creating a thread which reads from the ordering source, passes those messages to a block cutter, and writes the resulting block to the ledger.

6. **Halt()**: Frees the resources which were allocated to this chain. 

Here chain is not the (blockchain/channel), but an interface/object which is helping for the consensus of the channel. Object handling the consensus of a particular channel.  


### ConsenterSupport

```go
// ConsenterSupport provides the resources available to a Consenter implementation.
type ConsenterSupport interface {
	identity.SignerSerializer
	msgprocessor.Processor

	// VerifyBlockSignature verifies a signature of a block with a given optional
	// configuration (can be nil).
	VerifyBlockSignature([]*protoutil.SignedData, *cb.ConfigEnvelope) error

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

## New Packages: 
- `sync`
- `zap/zapcore`


