[https://hyperledger-fabric.readthedocs.io/en/release-2.5/dev-setup/devenv.html

Install SOFTHSM 
curl https://dist.opendnssec.org/source/softhsm-2.5.0.tar.gz --output softhsm-2.5.0.tar.gz

tar -xzf softhsm-2.5.0.tar.gz

cd  softhsm-2.5.0

./configure --disable-gost
make
sudo make install

https://wiki.opendnssec.org/display/SoftHSMDOCS/SoftHSM+Documentation+v2

softhsm2-util --init-token --slot 0 --label ForFabric --so-pin 1234 --pin 98765432


Build fabric :

make gotools


make basic-checks integration-test-prereqs
ginkgo -r ./integration/nwo

Build fabric:
make dist-clean all

consensus bdls.chain.go
](https://hyperledger-fabric.readthedocs.io/en/release-2.5/dev-setup/devenv.html

Install SOFTHSM 
curl https://dist.opendnssec.org/source/softhsm-2.5.0.tar.gz --output softhsm-2.5.0.tar.gz

tar -xzf softhsm-2.5.0.tar.gz

cd  softhsm-2.5.0

./configure --disable-gost
make
sudo make install

https://wiki.opendnssec.org/display/SoftHSMDOCS/SoftHSM+Documentation+v2

softhsm2-util --init-token --slot 0 --label ForFabric --so-pin 1234 --pin 98765432

export PKCS11_LIB="/usr/local/lib/softhsm/libsofthsm2.so"
export PKCS11_PIN=98765432
export PKCS11_LABEL="ForFabric"

Build fabric :

make gotools


make basic-checks integration-test-prereqs
ginkgo -r ./integration/nwo

Build fabric:
make dist-clean all

consensus bdls.chain.go)https://hyperledger-fabric.readthedocs.io/en/release-2.5/dev-setup/devenv.html

Install SOFTHSM 
curl https://dist.opendnssec.org/source/softhsm-2.5.0.tar.gz --output softhsm-2.5.0.tar.gz

tar -xzf softhsm-2.5.0.tar.gz

cd  softhsm-2.5.0

./configure --disable-gost
make
sudo make install

https://wiki.opendnssec.org/display/SoftHSMDOCS/SoftHSM+Documentation+v2

softhsm2-util --init-token --slot 0 --label ForFabric --so-pin 1234 --pin 98765432

export PKCS11_LIB="/usr/local/lib/softhsm/libsofthsm2.so"
export PKCS11_PIN=98765432
export PKCS11_LABEL="ForFabric"

Build fabric :

make gotools


make basic-checks integration-test-prereqs
ginkgo -r ./integration/nwo

Build fabric:
make dist-clean all

consensus bdls.chain.go
