# `submitC` channel
- `submitC` is a channel of type pointer to  `chan *submit`
- `startC` is the channel which closes when the node starts
- `doneC` is closed when the chain halts.
- When both the `startC` and `doneC` are open, then the chain is still running.


```go
type submit struct {
    req    *orderer.SubmitRequest
    leader chan uint64
}
```

### SubmitRequest
```go
// SubmitRequest wraps a transaction to be sent for ordering.
type SubmitRequest struct {
	Channel string `protobuf:"bytes,1,opt,name=channel,proto3" json:"channel,omitempty"`
	// last_validation_seq denotes the last
	// configuration sequence at which the
	// sender validated this message.

    // This is the configuration sequence. 
	LastValidationSeq uint64 `protobuf:"varint,2,opt,name=last_validation_seq,json=lastValidationSeq,proto3" json:"last_validation_seq,omitempty"`      
	// content is the fabric transaction
	// that is forwarded to the cluster member.
	Payload              *common.Envelope `protobuf:"bytes,3,opt,name=payload,proto3" json:"payload,omitempty"`
	XXX_NoUnkeyedLiteral struct{}         `json:"-"`
	XXX_unrecognized     []byte           `json:"-"`
	XXX_sizecache        int32            `json:"-"`
}

```

### Places it is used in: 
1. Initialized in `NewChain()` with empty channel
```go
submitC:           make(chan *submit),
```

2. WaitReady()

```go
// WaitReady blocks when the chain:
// - is catching up with other nodes using snapshot
//
// In any other case, it returns right away.
func (c *Chain) WaitReady() error {
	if err := c.isRunning(); err != nil {
		return err
	}

	select {
	case c.submitC <- nil:
	case <-c.doneC:
		return errors.Errorf("chain is stopped")
	}

	return nil
}

```

#### Why the `submitC <- nil` is done??

When submitC <- nil is inside a select statement, it will almost always be true, and this construct is typically used for non-blocking channel operations.

In Go, sending a value to an unbuffered channel (a channel with no buffer) is a blocking operation. It means that if there is no receiver ready to receive the sent value, the sender will be blocked until there is a receiver available to take the value.

However, when using a select statement, which is used to wait for multiple channel operations simultaneously, the case submitC <- nil: will be selected immediately if the channel is unbuffered because it can always accept the value (unless the channel has been explicitly closed).

Since the submitC <- nil operation is essentially non-blocking when used inside a select, it is commonly used for signaling purposes rather than actually passing meaningful data through the channel. The primary purpose of this construct is to trigger some behavior or communicate between goroutines without waiting for the other side to process the data.

The select statement in this case can wait on multiple channels simultaneously, including the submitC channel, without blocking on any of them. If the submitC channel is unbuffered, it will always be able to accept the nil value without blocking, and the corresponding case will be executed.

As a result, the purpose of using select in combination with submitC <- nil is to allow the WaitReady method to wait for either the submitC channel to accept the nil value or for the doneC channel to be closed, whichever event happens first. This construct provides a way to implement non-blocking synchronization and signaling between different parts of the codebase.





