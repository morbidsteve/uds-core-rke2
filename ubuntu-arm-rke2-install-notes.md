# Ubuntu 24.04 ARM Install Notes

My setup:

- Macbook Pro M3 with UTM installed
- Ubuntu server 24.04 (ARM)
- VM configured with:
  - 8 cores
  - 32GB RAM
- Single node RKE2 installation (server + agent combined)

**Note:** This is all done as root, `sudo su` to switch

## RKE 2 Setup

First, install RKE2:

```sh
curl -sfL https://get.rke2.io | sh -
systemctl enable rke2-server.service
systemctl start rke2-server.service
journalctl -u rke2-server -f
```

Now, download and install UDS:

```sh
wget https://github.com/defenseunicorns/uds-cli/releases/download/v0.27.7/uds-cli_v0.27.7_Linux_arm64
chmod +x uds-cli_v0.27.7_Linux_arm64
mv uds-cli_v0.27.7_Linux_arm64 /usr/local/bin/uds
```

Next, download and install zarf:

```sh
wget https://github.com/zarf-dev/zarf/releases/download/v0.56.0/zarf_v0.56.0_Linux_arm64
chmod +x zarf_v0.56.0_Linux_arm64
mv zarf_v0.56.0_Linux_arm64 /usr/local/bin/zarf
```

Update some sysctl settings:

```sh
sysctl -w fs.inotify.max_queued_events=16384
sysctl -w fs.inotify.max_user_instances=256
sysctl -w fs.inotify.max_user_watches=65536
```

Add rke2 deployed utils such as kubectl to `PATH`:

```sh
echo 'export PATH="/var/lib/rancher/rke2/bin:$PATH"' >> ~/.bashrc
```

```sh
Add `KUBECONFIG` to `PATH`:
```

echo 'export KUBECONFIG=/etc/rancher/rke2/rke2.yaml' >> ~/.bashrc

Source `.bashrc` so you don't need to start a new shell:

```sh
source ~/.bashrc
```

Now, using vim we're going to open `/etc/rancher/rke2/config.yaml` to add the following YAML. This expands cluster CIDR for temporary pods that setup the system, and disables `rke2-ingress-nginx` due to `istio` being used by UDS as well as `rke2-metrics-server` as UDS has its own:

Open the file with `vim /etc/rancher/rke2/config.yaml`.

Enter the following into that file:

```yaml
cluster-cidr: "10.42.0.0/16"  # Default - gives you ~65k IPs
disable:
    - rke2-ingress-nginx
    - rke2-metrics-server
```

Save and exit vim by typing `:x!` and pressing Enter.

## UDS Setup

We're doing this in the home directory (`~`), but follow these steps wherever you're comfortable.

Clone `uds-core-rke2` and create zarf packages:

```sh
git clone https://github.com/mcamick/uds-core-rke2.git
cd uds-core-rke2/uds-dev-stack
zarf package create --confirm --skip-sbom
cd ../minio-config
zarf package create --confirm --skip-sbom
cd ../

uds create --confirm
uds deploy uds-bundle-core-slim-dev-* --confirm
```
