

* curl -sSLO https://raw.githubusercontent.com/hyperledger/fabric/main/scripts/install-fabric.sh && chmod +x install-fabric.sh
* ./install-fabric.sh b s d

```
cd fabric-samples/test-network
./network.sh up createChannel -c channel1
docker ps -a
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go
```
## Clean up
```
docker rm -f $(docker ps -aq)
docker rmi -f $(docker images -q)
docker volume ls
docker volume prune
docker network prune
docker system prune -a --volumes

```
