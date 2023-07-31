# `consensus.Chain.Start()` Implementation
Function of Start: 
- Start should allocate whatever resources are needed for staying up to date with the chain.
- Typically, this involves creating a thread which reads from the ordering source, passes those
messages to a block cutter, and writes the resulting blocks to the ledger.


## Solo start implementation
```go
func (ch *chain) Start() {
	go ch.main()
}
```

