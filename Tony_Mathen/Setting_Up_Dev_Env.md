# Building Fabric


## Pre-requisites
Below are the steps to build fabric on a mac system. Ensure that you have homebrew, Go (version 1.20.6) and Docker installed and download the below two packages
- jq
- softhsm
```markdown
brew install jq softhsm
```

### Configuring softhsm
Initialize the token for softhsm and specify the library path (libsofthsm2.so), the PIN, and the label of your token.
```
softhsm2-util --init-token --slot 0 --label ForFabric --so-pin 1234 --pin 98765432
export PKCS11_LIB="/opt/homebrew/Cellar/softhsm/2.6.1/lib/softhsm/libsofthsm2.so"
export PKCS11_PIN=98765432
export PKCS11_LABEL="ForFabric"
```

### Install Development tools and verfiy the build environment
Clone the fabric repo in your developement system and run the following commands:
```
make gotools
make basic-checks integration-test-prereqs
ginkgo -r ./integration/nwo
```

### Building fabric
Build fabric using the below command
```
make dist-clean all
```
