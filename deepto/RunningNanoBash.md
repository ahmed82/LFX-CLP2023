# Test network - Nano bash

Test network Nano bash provides a set of minimal bash scripts to run a Fabric network on your local machine.

## Steps
1. Get installation script - `curl -sSLO https://raw.githubusercontent.com/hyperledger/fabric/main/scripts/install-fabric.sh && chmod +x install-fabric.sh`
2. `./install-fabric.sh samples binary`
3. To run the chaincode as a service, we need to configure the peer to use the `ccaas` external builder downloaded with the binaries above. The path specified in the default `fabric-samples/config/core.yaml` file is only valid within the peer container which we won't be using. Edit it and modify the `externalBuilders` field to point to the correct path (location to `fabric-samples/builders/ccaas`):
    ```
    externalBuilders:
        - name: ccaas_builder
            path: /workspace/fabric-workspace/fabric-samples/builders/ccaas  //Replace with path to 
            propagateEnvironment:
            - CHAINCODE_AS_A_SERVICE_BUILDER_CONFIG
    ```
4. Starting the network (each component separately)
    * Open terminal windows for 3 ordering nodes or 4 if running BFT Consensus, 4 peer nodes, and 4 peer admins
    *  `cd fabric-samples/test-network-nano-bash` in each terminals
    * In the first orderer terminal, run `./generate_artifacts.sh BFT` to generate crypto material (calls cryptogen) and application channel genesis block and configuration transactions (calls configtxgen). The artifacts will be created in the crypto-config and channel-artifacts directories.
        * `fabric-samples/test-network-nano-bash/bft-config/configtx.yaml` file used
        * Getting error 
         ![Alt text](<assets/Screenshot from 2023-08-02 07-56-54-1.png>)
        * I think the problem is the install command fetches version 2.5's binary, need to figure out how to get it from the main branch 
