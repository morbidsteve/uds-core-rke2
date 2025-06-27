<!-- My setup is Macbook Pro M3, with UTM installed -->
<!-- Install Ubuntu server 24.04 (obviously it's arm) -->
<!-- Setup VM with 8 cores, 32GB ram -->
<!-- Install RKE2, just one piece, so the one ubuntu machine is both the server and and agent, literally just run the commands -->
curl -sfL https://get.rke2.io | sh -
systemctl enable rke2-server.service
systemctl start rke2-server.service
journalctl -u rke2-server -f



#<!-- Download / install UDS -->
wget https://github.com/defenseunicorns/uds-cli/releases/download/v0.27.7/uds-cli_v0.27.7_Linux_arm64
chmod +x uds-cli_v0.27.7_Linux_arm64
mv uds-cli_v0.27.7_Linux_arm64 /usr/local/bin/uds

#<!-- Download / install zarf -->
wget https://github.com/zarf-dev/zarf/releases/download/v0.56.0/zarf_v0.56.0_Linux_arm64
chmod +x zarf_v0.56.0_Linux_arm64
mv zarf_v0.56.0_Linux_arm64 /usr/local/bin/zarf



<!-- sysctl changes -->
sysctl -w fs.inotify.max_queued_events=16384
sysctl -w fs.inotify.max_user_instances=256
sysctl -w fs.inotify.max_user_watches=65536

<!-- Add rke2 deployed utils such as kubectl -->
echo 'export PATH="/var/lib/rancher/rke2/bin:$PATH"' >> ~/.bashrc
<!-- Add kubeconfig to path -->
echo 'export KUBECONFIG=/etc/rancher/rke2/rke2.yaml' >> ~/.bashrc
source ~/.bashrc



#<!-- Create file: /etc/rancher/rke2/config.yaml, expand cluster cidr for temp pods that set things up, and disable ingress due to istio instead, and uds has it's own metrics server -->
cluster-cidr: "10.42.0.0/16"  # Default - gives you ~65k IPs
disable:
    - rke2-ingress-nginx
    - rke2-metrics-server


git clone https://github.com/mcamick/uds-core-rke2.git
cd uds-core-rke2
cd uds-dev-stack/
zarf package create --confirm --skip-sbom
cd ../

cd minio-config
zarf package create --confirm --skip-sbom
cd ../

uds create --confirm
uds deploy uds-bundle-core-slim-dev-* --confirm