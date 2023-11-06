# HandleMessage in SmartBFT

```go
// HandleMessage handles the message from the sender
func (c *BFTChain) HandleMessage(sender uint64, m *smartbftprotos.Message) {
	c.Logger.Debugf("Message from %d", sender)

	c.consensus.HandleMessage(sender, m)
}
```

The `c.consensus.HandleMessage(sender, m)` is:
```go
func (c *Consensus) HandleMessage(sender uint64, m *protos.Message) {
	// eacgh node has a map of nodes --> will have to check here: !@#
	if _, exists := c.nodeMap.Load(sender); !exists {
		c.Logger.Warnf("Received message from unexpected node %d", sender)
		return
	}
	c.consensusLock.RLock()
	defer c.consensusLock.RUnlock()
	c.controller.ProcessMessages(sender, m)
}
```

### ProcessMessages:
```go
// ProcessMessages dispatches the incoming message to the required component
func (c *Controller) ProcessMessages(sender uint64, m *protos.Message) {
	c.Logger.Debugf("%d got message from %d: %s", c.ID, sender, MsgToString(m))
	switch m.GetContent().(type) {
	case *protos.Message_PrePrepare, *protos.Message_Prepare, *protos.Message_Commit:
		c.currViewLock.RLock()
		view := c.currView
		c.currViewLock.RUnlock()
		view.HandleMessage(sender, m)
		c.ViewChanger.HandleViewMessage(sender, m)
		if sender == c.leaderID() {
			c.LeaderMonitor.InjectArtificialHeartbeat(sender, c.convertViewMessageToHeartbeat(m))
		}
	case *protos.Message_ViewChange, *protos.Message_ViewData, *protos.Message_NewView:
		c.ViewChanger.HandleMessage(sender, m)
	case *protos.Message_HeartBeat, *protos.Message_HeartBeatResponse:
		c.LeaderMonitor.ProcessMsg(sender, m)
	case *protos.Message_StateTransferRequest:
		c.respondToStateTransferRequest(sender)
	case *protos.Message_StateTransferResponse:
		c.Collector.HandleMessage(sender, m)
	default:
		c.Logger.Warnf("Unexpected message type, ignoring")
	}
}
```

### HandleMessage handles incoming messages:
```go
// HandleMessage handles incoming messages
func (v *View) HandleMessage(sender uint64, m *protos.Message) {
	msg := &incMsg{sender: sender, Message: m}
	select {
	case <-v.abortChan:
		return
	case v.incMsgs <- msg:
	}
}
```

The HandleMessage method is responsible for handling incoming messages in the context of a View type. It creates a message struct, and depending on the state of the abortChan, it either sends the message to the incMsgs channel for further processing or returns immediately if an abort signal is detected. 

--> I can check out the `incMsgs`

### View is responsible for running the view protocol: 
// View is responsible for running the view protocol

From `incMsgs` it is taken out from: 
```go
func (v *View) run() {
	defer v.viewEnded.Done()
	defer func() {
		v.ViewSequences.Store(ViewSequence{
			ProposalSeq: v.ProposalSequence,
			ViewActive:  false,
		})
	}()
	for {
		select {
		case <-v.abortChan:
			return
		case msg := <-v.incMsgs:
			v.processMsg(msg.sender, msg.Message)
		default:
			v.doPhase()
		}
	}
}
```

`v.processMsg`:

```go
func (v *View) processMsg(sender uint64, m *protos.Message) {
	if v.Stopped() {
		return
	}
	// Ensure view number is equal to our view
	msgViewNum := viewNumber(m)
	msgProposalSeq := proposalSequence(m)

	if msgViewNum != v.Number {
		v.Logger.Warnf("%d got message %v from %d of view %d, expected view %d", v.SelfID, m, sender, msgViewNum, v.Number)
		if sender != v.LeaderID {
			v.discoverIfSyncNeeded(sender, m)
			return
		}
		v.FailureDetector.Complain(v.Number, false)
		// Else, we got a message with a wrong view from the leader.
		if msgViewNum > v.Number {
			v.Sync.Sync()
		}
		v.stop()
		return
	}

	if msgProposalSeq == v.ProposalSequence-1 && v.ProposalSequence > 0 {
		v.handlePrevSeqMessage(msgProposalSeq, sender, m)
		return
	}

	v.Logger.Debugf("%d got message %s from %d with seq %d", v.SelfID, MsgToString(m), sender, msgProposalSeq)
	// This message is either for this proposal or the next one (we might be behind the rest)
	if msgProposalSeq != v.ProposalSequence && msgProposalSeq != v.ProposalSequence+1 {
		v.Logger.Warnf("%d got message from %d with sequence %d but our sequence is %d", v.SelfID, sender, msgProposalSeq, v.ProposalSequence)
		v.discoverIfSyncNeeded(sender, m)
		return
	}

	msgForNextProposal := msgProposalSeq == v.ProposalSequence+1

	if pp := m.GetPrePrepare(); pp != nil {
		v.processPrePrepare(pp, m, msgForNextProposal, sender)
		return
	}

	// Else, it's a prepare or a commit.
	// Ignore votes from ourselves.
	if sender == v.SelfID {
		return
	}

	if prp := m.GetPrepare(); prp != nil {
		if msgForNextProposal {
			v.nextPrepares.registerVote(sender, m)
		} else {
			v.prepares.registerVote(sender, m)
		}
		return
	}

	if cmt := m.GetCommit(); cmt != nil {
		if msgForNextProposal {
			v.nextCommits.registerVote(sender, m)
		} else {
			v.commits.registerVote(sender, m)
		}
		return
	}
}

```

Three types: PrePrepare, Prepare, Commit




# Now For Raft: 

- No handleMessage found specifically for `raft`, but `handleMessage` was there for `gossip` (which I don't think we need to tackle).

# For BDLS:
`receiveMessage`: Receives the consensus message and processes them, based on different types of messages.

`handleConsensusMessage`: 