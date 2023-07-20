# Setting up Fabric Test Network 
> Machine Specifications: Macbook Air M1 (MacOS 13.4.1 (c))

## Prerequisites
Followed [this](https://hyperledger-fabric.readthedocs.io/en/release-2.5/prereqs.html) documentation page and was able to successfully install all the prerequisites. 
- Homebrew version: `4.0.29`
- Git: `2.38.0`
- cURL: `7.88.1 (x86_64-apple-darwin22.0)`
- Docker: `20.10.23, build 7155243`
- Docker Compose: `v2.15.1`
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
./install-fabric.sh docker samples binary
```

## Running Fabric Test Network
- Had Docker Desktop version 4.17, so had to install `2.5.0.1` from [here](https://docs.docker.com/desktop/previous-versions/2.x-mac/).

