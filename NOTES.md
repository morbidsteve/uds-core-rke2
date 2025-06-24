# Installation notes for local VM and RKE2 on Ubuntu 22.04

## Disabling RKE2 components that conflict with UDS functional layers
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

## Sysctl changes
The defautl sysctl settings for fs.inotify.* are insufficient when deploying all functional layers.

You can see the defaults by running `sudo sysctl fs.inotify`

To set higher limits on these, run:
```
sudo sysctl -w fs.inotify.max_queued_events = 16384
sudo sysctl -w fs.inotify.max_user_instances = 256
sudo sysctl -w fs.inotify.max_user_watches = 65536
```

## SSO itegration with doom
The `ghcr.io/uds-packages/dos-games` Package, which is not public, is started in ambient mode by default, which does NOT support authervice integration.  To update the Package CR to get SSO configuration running, you will need to update the Package CR in the `dos-games` namespace.

1. Start by getting the current Package CR definition: `kubectl get package -n dos-games dos-games -o yaml > ./dos-games-package.yaml`
1. That should result in the `dos-games-package.yaml` looking something like:
```yaml
apiVersion: uds.dev/v1alpha1
kind: Package
metadata:
  name: dos-games
  namespace: dos-games
spec:
  network:
    allow:
    - direction: Ingress
      remoteGenerated: IntraNamespace
    - direction: Egress
      remoteGenerated: IntraNamespace
    expose:
    - gateway: tenant
      host: doom
      podLabels:
        app: game
      port: 8000
      service: doom
    serviceMesh:
      mode: ambient
```

1. Update it to add the SSO configurations and change the `serviceMesh.mode` to `sidecar`
```yaml
apiVersion: uds.dev/v1alpha1
kind: Package
metadata:
  name: dos-games
  namespace: dos-games
spec:
  network:
    allow:
    - direction: Ingress
      remoteGenerated: IntraNamespace
    - direction: Egress
      remoteGenerated: IntraNamespace
    expose:
    - gateway: tenant
      host: doom
      podLabels:
        app: game
      port: 8000
      service: doom
    serviceMesh:
      mode: sidecar
  sso:
  - alwaysDisplayInConsole: false
    clientId: uds-core-doom
    enableAuthserviceSelector:
      app: game
    enabled: true
    groups:
      anyOf:
      - /UDS Core/Admin
    name: Doom
    protocolMappers: []
    publicClient: false
    redirectUris:
    - https://doom.uds.dev/home
    serviceAccountsEnabled: false
    standardFlowEnabled: true
```
>Note that the redirectURI specified has a `/home` added to it.  This isn't technically a path, but all requests to the virtual service are redirected to `/`, and having a root path as the redirect uri is forbidden in Keycloak
