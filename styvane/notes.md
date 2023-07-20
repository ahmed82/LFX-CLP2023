- Create a new directory fabric 
  ```bash
  mkdir fabric && cd fabric
  ```
  
- Get the installation script
  ```bash
  curl -sSLO https://raw.githubusercontent.com/hyperledger/fabric/main/scripts/install-fabric.sh && chmod +x install-fabric.sh
  ```
- Install components
  ```
  ./install-fabric.sh docker samples binary
  ```
  
- Run test network
  - Go to test-network directory `cd fabric-samples/test-network`
  - Run `./network up`

- Check the instances state by running `docker ps`

- Create a new channel using `./network.sh createChannel -c s1`

- Deploy a chaincode on the newly created channel `s1`
  ```bash
  ./network.sh deployCC -ccn basic -c s1 -ccp ../asset-transfer-basic/chaincode-go -ccl go
  ```
 
- Export envars by sourcing export.sh from the test-network directory.

- Initialize the asset on the `s1` channel.
  ```bash
  peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C s1 -n basic --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" -c '{"function":"InitLedger","Args":[]}'
  ```

- Query the ledger.
  ```bash
  peer chaincode query -C s1 -n basic -c '{"Args":["GetAllAssets"]}'
  ```



  
  


