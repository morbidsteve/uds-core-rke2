<<<<<<< HEAD
# UDS Core Slim RKE2 

UDS Core Slim Dev that is not opinionated around k3d.

Once your vanilla cluster is up and running, run the following commands:

```shell
cd uds-dev-stack/
zarf package create --confirm --skip-sbom
cd ../

uds create --confirm
uds deploy uds-bundle-core-slim-dev-* --confirm
```
