1. cd fabric-samples/bin
2. mkdir ../test-config && touch ../test-config/crypto-config.yaml
2. Generate certificates and MSPs with cryptogen:
    * Get sample configuration :   
    ./cryptogen showtemplate > ../test-config/crypto-config.yaml
    * Open crypto-config.yaml and change the configuration to your needs
    * Generate the crypto mateial :  
     ./cryptogen generate  --config=../test-config/crypto-config.yaml --output=../test-config/
    * Now we can see ordererOrganizations and peerOrganizations within test-config
3. Create genesis block with configtxgen