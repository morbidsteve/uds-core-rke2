# UDS Core Slim RKE2 

## Installation notes for local VM and RKE2 on Ubuntu 22.04

### Disabling RKE2 components that conflict with UDS functional layers
There are two services that are started by RKE2 by default that should be disabled to avoid conflicts or failed package deployments:
1. `rke2-ingress-nginx` - provides an Ingress service that could interfere with Istio and MetalLB
1. `rke2-metrics-server`- provides an instance of Kubernetes Metrics Server.  Can be kept, but if deploying the `core-metrics-server` package, you will want to remove it first

Removal of these components can be accomplished pre-installation of RKE2 by creating a file at `/etc/rancher/rke2/config.yaml` with the following contents:
```
disable:
  - rke2-ingress-nginx
  - rke2-metrics-server
```

If RKE2 is already installed, you can add this file and then restart the rke2-server service with `sudo systemctl restart rke2-server`.  

This should prevent/cleanup the components.

### Sysctl changes
The defautl sysctl settings for fs.inotify.* are insufficient when deploying all functional layers.

You can see the defaults by running `sudo sysctl fs.inotify`

To set higher limits on these, run:
```
sudo sysctl -w fs.inotify.max_queued_events=16384
sudo sysctl -w fs.inotify.max_user_instances=256
sudo sysctl -w fs.inotify.max_user_watches=65536
```

### UDS Core for RKE Experiment Installation Instructions

Once your RKE2 cluster is up and running, run the following commands:

```shell
cd uds-dev-stack/
zarf package create --confirm --skip-sbom
cd ../

cd minio-config
zarf package create --confirm --skip-sbom
cd ../

uds create --confirm
uds deploy uds-bundle-core-slim-dev-* --confirm
```
