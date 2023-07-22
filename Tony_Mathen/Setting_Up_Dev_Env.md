# Docker Network overlaps with the host address

### Facing this issue when building fabric
```
make dist-clean all
```
Error Message:
WARNING: docker network 172.20.0.0/16 overlaps with host address 172.20.206.165.
Docker containers may not have connectivity causing chaincode registration to fail with 'no route to host'.
