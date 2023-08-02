# Analysis of etcd Raft package

Raft is a consensus algorithm designed for distributed systems to achieve fault-tolerance and data consistency. Consensus algorithms ensure that all nodes in a distributed system agree on the same sequence of actions, even in the presence of failures.

The Raft consensus algorithm is based on three key components:

1.  Leader Election: Nodes participate in leader elections to choose a single leader responsible for managing the cluster's operations.
2.  Log Replication: The leader appends log entries to its log and replicates them to followers. Followers then apply these log entries to their state machines, ensuring consistency.
3.  Safety: Raft ensures safety by electing a leader through a majority vote and committing log entries only when a majority of nodes acknowledge their replication. This ensures that all committed log entries are durably stored and will eventually be applied to all nodes.

### Existing Solutions
1. Blockchain - Proof of Work
2. Paxos - Difficult to implement

### Desired Properties
1. Safety - At most one winner per term
2. Liveliness - Somebody eventually wins

### Election Outcomes
1. Receive Majority  - Become leader, tell everyone else
2. Election times out -  Keep being a candidate. Hold new election

### Log Replication
1. Log entry - (index, term, data) tuple
2. Entry is committed if it is on a majority of nodes

### Safe Replication
Followers wont vote for bad candidates
Log coherency → Deny votes if the candidate has less complete log
-   Don’t vote if your term is better than candidate’s
-   Don’t vote if your term is the same but your index is better than candidate’s

###  Safe log commitment
An entry in a log is committed if
1. It is majority stored and 
2. At least one entry from leader’s term is also majority stored

### Struct types in raft
Some of the types in the raft package are as follows:
1. ConfState
```go
type ConfState struct {
	// The voters in the incoming config. (If the configuration is not joint,
	// then the outgoing config is empty).
	Voters []uint64 `protobuf:"varint,1,rep,name=voters" json:"voters,omitempty"`
	// The learners in the incoming config.
	Learners []uint64 `protobuf:"varint,2,rep,name=learners" json:"learners,omitempty"`
	// The voters in the outgoing config.
	VotersOutgoing []uint64 `protobuf:"varint,3,rep,name=voters_outgoing,json=votersOutgoing" json:"voters_outgoing,omitempty"`
	// The nodes that will become learners when the outgoing config is removed.
	// These nodes are necessarily currently in nodes_joint (or they would have
	// been added to the incoming config right away).
	LearnersNext []uint64 `protobuf:"varint,4,rep,name=learners_next,json=learnersNext" json:"learners_next,omitempty"`
	// If set, the config is joint and Raft will automatically transition into
	// the final config (i.e. remove the outgoing config) when this is safe.
	AutoLeave bool `protobuf:"varint,5,opt,name=auto_leave,json=autoLeave" json:"auto_leave"`
}
```
2. Message
```go
type Message struct {
	Type MessageType `protobuf:"varint,1,opt,name=type,enum=raftpb.MessageType" json:"type"`
	To   uint64      `protobuf:"varint,2,opt,name=to" json:"to"`
	From uint64      `protobuf:"varint,3,opt,name=from" json:"from"`
	Term uint64      `protobuf:"varint,4,opt,name=term" json:"term"`
	// logTerm is generally used for appending Raft logs to followers. For example,
	// (type=MsgApp,index=100,logTerm=5) means leader appends entries starting at
	// index=101, and the term of entry at index 100 is 5.
	// (type=MsgAppResp,reject=true,index=100,logTerm=5) means follower rejects some
	// entries from its leader as it already has an entry with term 5 at index 100.
	LogTerm    uint64   `protobuf:"varint,5,opt,name=logTerm" json:"logTerm"`
	Index      uint64   `protobuf:"varint,6,opt,name=index" json:"index"`
	Entries    []Entry  `protobuf:"bytes,7,rep,name=entries" json:"entries"`
	Commit     uint64   `protobuf:"varint,8,opt,name=commit" json:"commit"`
	Snapshot   Snapshot `protobuf:"bytes,9,opt,name=snapshot" json:"snapshot"`
	Reject     bool     `protobuf:"varint,10,opt,name=reject" json:"reject"`
	RejectHint uint64   `protobuf:"varint,11,opt,name=rejectHint" json:"rejectHint"`
	Context    []byte   `protobuf:"bytes,12,opt,name=context" json:"context,omitempty"`
}
```
3. Snapshot
```go
type Snapshot struct {
	Data     []byte           `protobuf:"bytes,1,opt,name=data" json:"data,omitempty"`
	Metadata SnapshotMetadata `protobuf:"bytes,2,opt,name=metadata" json:"metadata"`
}
```
4. HardState
```go
type HardState struct {
	Term   uint64 `protobuf:"varint,1,opt,name=term" json:"term"`
	Vote   uint64 `protobuf:"varint,2,opt,name=vote" json:"vote"`
	Commit uint64 `protobuf:"varint,3,opt,name=commit" json:"commit"`
}
```
