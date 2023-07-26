# LFX-CLP2023
`Protocols in fabric references`
* https://github.com/hyperledger-labs/mirbft

* https://github.com/SmartBFT-Go/fabric

* Fabric RFC for SmartBFT: https://github.com/hyperledger/fabric-rfcs/pull/33/commits/433095b29588e4df894a829f5655b3647666a233

* That was proposed, and read the follow-up discussion, https://github.com/hyperledger/fabric-rfcs/pull/33

### NOTE :raising_hand:
Requirement for the 07-26-27 session: to implement the Start function in bdls/chain.go:
```go
func (c *Chain) Start() {
	//TODO
}
```
Please review the following for reference:
## Raft implementation in Fabric

* [FAB-11162] Implement bare minimum Raft-based chain
https://github.com/BDLS-bft/fabric/commit/7f12d1b3ae31bc6f76b8aba376afeaca93e5b345

* [FAB-11833] Say hello to Raft OSN
https://github.com/BDLS-bft/fabric/commit/96735d2fb22abefadc8bddddb85199bb140b2a5d

* [FAB-13178] Move raft logic to its own file
https://github.com/BDLS-bft/fabric/commit/fc7395f45ffc32683c0a25f74eb3eee76e59fb9d

* Raft in the chain.go as of today's code Start() function
## BFT start protocol 
* Smart BFT-go chain.go - Start() function.
https://github.com/SmartBFT-Go/fabric
## How to start bdls in:
* BDLS bdls.NewConsensus(config)
* BDLSChain

## Certificates Management Guide
https://hyperledger-fabric.readthedocs.io/en/release-2.5/certs_management.html

## Membership Service Provider
https://hyperledger-fabric.readthedocs.io/en/release-2.5/membership/membership.html

## etcd RAFT
https://github.com/etcd-io/raft
