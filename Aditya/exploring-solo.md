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
	normalMsg *cb.Envelope // normal trasactions
	configMsg *cb.Envelope // transacitons which update the configuration.
}
```






## New Packages: 
- `sync`
- `zap/zapcore`


