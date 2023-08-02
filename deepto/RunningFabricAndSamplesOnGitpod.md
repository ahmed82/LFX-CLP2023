# Setting up Fabric and Samples using Gitpod

## Setting up a project on Gitpod
1. Make a new repository on Github e.g https://github.com/deepto98/fabric-workspace
2. Login on [Gitpod](https://gitpod.io) using your Github
3. Go to New Workspace. In the Select Repository dropdown, choose the Github repo you created. Click on Continue. Now a VSCode instance should open up in the browser, inside which you can code.

## Running Fabric and Samples
1. Check the Linux section in the [prerequisites](https://hyperledger-fabric.readthedocs.io/en/release-2.5/prereqs.html) documentation. These should be avaiable by default in gitpod, but verify them just to be sure.
2. [Install Fabric and Fabric Samples](https://hyperledger-fabric.readthedocs.io/en/release-2.5/install.html):
    * Inside Gitpod, download the Samples install script  
    `curl -sSLO https://raw.githubusercontent.com/hyperledger/fabric/main/scripts/install-fabric.sh && chmod +x install-fabric.sh`
    * Install the docker, samples and binary components with  
    `./install-fabric.sh docker samples binary`
    * Now the docker images for the `peer, orderer, ccenv, tools` should be avaialbe on the system, and you can check them by running  `docker images`
3. [Running a Fabric Test Network](https://hyperledger-fabric.readthedocs.io/en/release-2.5/test_network.html):  
    **Note : I've added the main steps here from the docs, for detailed steps and documentation, visit https://hyperledger-fabric.readthedocs.io/en/release-2.5/test_network.html**

    * Bring up the test network : 
        ```
        cd fabric-samples/test-network
        
        ./network.sh down // Cleanup containers or artifacts from any previous runs
        # TODO - add all cleanup steps shared by Ahmed 

        ./network.sh up // Creates a Fabric network that consists of two peer nodes, one ordering node, without a channel

        ./network.sh up createChannel // Creates a network with a channel
        ```
    * Verify if the docker containers for the peers and orderer are active : `docker ps -a`
    * Add a channel 
        ```
        ./network.sh createChannel // creates a channel named mychannel
        # or specify the channel name with
        ./network.sh createChannel -c channel1
        ```
    * Starting a chaincode on the channel:
        ```
        ./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go

        # To add a channel name : -c channel1
        ```
        This installs the asset-transfer-basic chaincode on the peers and deploys the chaincode on  the specified channel (mychannel by default). The smart contract code can be found in `fabric-samples/asset-transfer-basic/chaincode-go/smartcontract.go`
    * Setup Peer CLI to for interacting with the network
        ```
        # 1. cd to the  fabric-samples/test-network dir if not there already

        # 2. The binaries for the peer CLI are in fabric-samples/bin, add them to  PATH

        export PATH=${PWD}/../bin:$PATH

        # 3. Set FABRIC_CFG_PATH to point to the core.yaml file in the fabric-samples repository

        export FABRIC_CFG_PATH=$PWD/../config/

        # TODO - verify this command, since it doesn't exactly point to the yaml file (fabric-samples/config/core.yaml)

        # 4. Set the environment variables that allow you to operate the peer CLI as Org1:

        export CORE_PEER_TLS_ENABLED=true
        export CORE_PEER_LOCALMSPID="Org1MSP"
        export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
        export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
        export CORE_PEER_ADDRESS=localhost:7051

        ```
    * Interacting with the network using the Peer CLI
        * Initialize the ledger with assets  
            ```
            peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" -c '{"function":"InitLedger","Args":[]}'

            ```
        * Query the Ledger:
            ```
            peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'

            ```
        * Transfer ownership of an asset
            ```
            peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" -c '{"function":"InitLedger","Args":[]}'

            ```
        * Switch to Org2 to to query the chaincode running on the Org2 peer:
            ```
            # Environment variables for Org2

            export CORE_PEER_TLS_ENABLED=true
            export CORE_PEER_LOCALMSPID="Org2MSP"
            export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
            export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
            export CORE_PEER_ADDRESS=localhost:9051
            ```
        * Query the asset-transfer (basic) chaincode running on peer0.org2.example.com:
            ```
            peer chaincode query -C mychannel -n basic -c '{"Args":["ReadAsset","asset6"]}'

            ```
    * Bringing down the network - `./network.sh down`