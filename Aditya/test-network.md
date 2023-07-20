# Setting up Fabric Test Network 
> Machine Specifications: Macbook Air M1 (MacOS 13.4.1 (c))

## Prerequisites
Followed [this](https://hyperledger-fabric.readthedocs.io/en/release-2.5/prereqs.html) documentation page and was able to successfully install all the prerequisites. \\

**Versions**:
- Homebrew: `4.0.29`
- Git: `2.38.0`
- cURL: `7.88.1 (x86_64-apple-darwin22.0)`
- Docker Desktop: `4.21.1 (114176)`
- Docker: `24.0.2, build cb74dfc`
- Docker Compose: `v2.19.1`
- Go: `go1.20.6 darwin/arm64`
- jq: `jq-1.6`

## Download Fabric samples, Docker images, and binaries
1. Created a working directory
```zsh
mkdir -p $HOME/go/src/github.com/sadityakumar9211
```
2. cURL to get the install script from GitHub
```zsh
curl -sSLO https://raw.githubusercontent.com/hyperledger/fabric/main/scripts/install-fabric.sh && chmod +x install-fabric.sh
```
3. Verifying whether the script was downloaded, with -h flag
```zsh
./install-fabric.sh -h
```
4. Installed the latest fabric docker images, fabric-binaries and fabric-samples
```zsh
./install-fabric.sh d s b
```

## Running Fabric Test Network
I am running the latest Docker Desktop version for MacOS (4.21.1 (114176)).
### Bringing up the test network
1. Used the `network.sh` script, to bring the fabric test network, two peers, and one orderer up.
```zsh
./network.sh up
```
It created 4 docker containers which I verified using `docker ps -a`: 
```zsh
CONTAINER ID   IMAGE                               COMMAND             CREATED         STATUS         PORTS                                                                    NAMES
fca13e7a3ad3   hyperledger/fabric-tools:latest     "/bin/bash"         6 minutes ago   Up 6 minutes                                                                            cli
3c6338d1d1b3   hyperledger/fabric-peer:latest      "peer node start"   6 minutes ago   Up 6 minutes   0.0.0.0:7051->7051/tcp, 0.0.0.0:9444->9444/tcp                           peer0.org1.example.com
7132a2fdc982   hyperledger/fabric-peer:latest      "peer node start"   6 minutes ago   Up 6 minutes   0.0.0.0:9051->9051/tcp, 7051/tcp, 0.0.0.0:9445->9445/tcp                 peer0.org2.example.com
4dc4e736083d   hyperledger/fabric-orderer:latest   "orderer"           6 minutes ago   Up 6 minutes   0.0.0.0:7050->7050/tcp, 0.0.0.0:7053->7053/tcp, 0.0.0.0:9443->9443/tcp   orderer.example.com
```

### Creating a Channel
Created a channel with name `ch-adi-1` between Org1 and Org2 and their peers join the network. Without passing the channel name it created a channel with name `mychannel` 
```zsh
./network.sh createChannel -c ch-adi-1
```

### Starting a chaincode on a channel
Installed `asset-transfer-basic` chaincode on peers0 of both Org1 and Org2 and deployed the chaincode on channel `ch-adi-1`.
```zsh
./network.sh deployCC -c ch-adi-1 -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go
```

### Interacting with the network
- Setting up the binaries to the CLI Path and `FABRIC_CFG_PATH`. Edited the `~/.zshrc` file.
```shell
export PATH=${PWD}/../bin:$PATH
export FABRIC_CFG_PATH=${PWD}/../config/
```
- Setting up environment variables so that `peer` will refer to Org1 peer.
```zsh
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
```

- Initializing the ledger with assets
```zsh
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C ch-adi-1 -n basic --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" -c '{"function":"InitLedger","Args":[]}'
```
I got output: 
```zsh
2023-07-20 19:55:44.851 IST 0001 INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200
```
- Getting the list of assests that were added by the above transaction
```zsh
peer chaincode query -C ch-adi-1 -n basic -c '{"Args":["GetAllAssets"]}'
```
Got output:
```zsh
[{"AppraisedValue":300,"Color":"blue","ID":"asset1","Owner":"Tomoko","Size":5},
{"AppraisedValue":400,"Color":"red","ID":"asset2","Owner":"Brad","Size":5},
{"AppraisedValue":500,"Color":"green","ID":"asset3","Owner":"Jin Soo","Size":10},{"AppraisedValue":600,"Color":"yellow","ID":"asset4","Owner":"Max","Size":10},{"AppraisedValue":700,"Color":"black","ID":"asset5","Owner":"Adriana","Size":15},{"AppraisedValue":800,"Color":"white","ID":"asset6","Owner":"Michel","Size":15}]
```

- Transferring the ownership of asset 6 to Christopher:
```zsh
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C ch-adi-1 -n basic --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" -c '{"function":"TransferAsset","Args":["asset6","Christopher"]}'
```

I got output: 
```zsh
2023-07-20 20:06:49.608 IST 0001 INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200 payload:"Michel"
```

- Querying the chaincode installed on the peer of Org2:
Setting up environment variables to act as Org2
```zsh
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
```

- Querying the chaincode on peer of Org2:
```zsh
peer chaincode query -C ch-adi-1 -n basic -c '{"Args":["ReadAsset","asset6"]}'
```
I got the output as: 
```shell
{"AppraisedValue":800,"Color":"white","ID":"asset6","Owner":"Christopher","Size":15}
```

## Bring down the network
```zsh
./network.sh down
```
