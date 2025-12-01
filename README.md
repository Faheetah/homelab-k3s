# Setup Server

*Note: this install process mirrors the script executable from `https://get.k3s.io/` noted in the k3s quick start[^1], it is included as explicit steps to demonstrate the full install process. For other installations such as node setup, use the get.k3s.io script.*

## Install k3s

```bash
sudo wget -O /usr/local/bin/k3s https://github.com/k3s-io/k3s/releases/download/v1.34.1%2Bk3s1/k3s
sudo chmod +x /usr/local/bin/k3s
k3s completion bash | sudo tee /etc/bash_completion.d/k3s
```

## Set System Options

Uncomment `net.ipv4.ip_forward=1` in `/etc/sysctl.conf` then run `sudo sysctl --system`.

## Add to systemd

Create the systemd service file.

<sub>/etc/systemd/system/k3s.service</sub>
```ini
[Unit]
Description=Lightweight Kubernetes
Documentation=https://k3s.io
Wants=network-online.target
After=network-online.target

[Install]
WantedBy=multi-user.target

[Service]
Type=notify
EnvironmentFile=-/etc/default/%N
EnvironmentFile=-/etc/sysconfig/%N
EnvironmentFile=-/etc/systemd/system/k3s.env
KillMode=process
Delegate=yes
User=root
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=1048576
LimitNPROC=infinity
LimitCORE=infinity
TasksMax=infinity
TimeoutStartSec=0
Restart=always
RestartSec=5s
ExecStartPre=-/sbin/modprobe br_netfilter
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/local/bin/k3s server
```

Enable and start k3s server.

```bash
sudo systemctl enable k3s
sudo systemctl start k3s
```

## Create Symlinks to k3s Subcommands (optional)

k3s uses $0 to determine the executing binary path, which lets it interpret a symlink for kubectl as running the kubectl command. Also setup bash completion for kubectl. crictl also has bash completion, but ctr does not.

```bash
sudo ln -fs /usr/local/bin/k3s /usr/local/bin/kubectl
kubectl completion bash | sudo tee /etc/bash_completion.d/kubectl
sudo ln -fs /usr/local/bin/k3s /usr/local/bin/crictl
sudo ln -fs /usr/local/bin/k3s /usr/local/bin/ctr
```


# Setup kubectl

kubectl can be installed anywhere to interact with the cluster, such as a personal workstation[^2].

```bash
sudo wget -O /usr/local/bin/kubectl "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo chmod +x /usr/local/bin/kubectl
kubectl completion bash | sudo tee /etc/bash_completion.d/kubectl
```

Copy the `/etc/rancher/k3s/k3s.yaml` file from the node controller to the local machine at ~/.kube/config, replacing the `clusters[0].cluser.server` section with the remote node's URL. It is advised to ensure the permissions for the folder and config file be restricted only to the local user with 0600.


# Install Agent Nodes (optional)

Get the node token from `/var/lib/rancher/k3s/server/node-token` on the control node. Run the get.k3s.io install script on the agent using the node token from the control node and the address of the control node API endpoint.

```bash
curl -sfL https://get.k3s.io | K3S_URL=https://myserver:6443 K3S_TOKEN=mynodetoken sh -
```


# Troubleshooting

## Certificate Errors for kubectl

k3s will setup certificates using the system hostname[^3]. To add additional SANs to the cerficiate, add the following to *config.yaml* (not *k3s.yaml*).

<sub>/etc/rancher/k3s/config.yaml</sub>
```yaml
tls-san:
  - "myserverfqdn"
```

## Manifest Issues

Ensure the correct apiVersion[^4] is being used for the resource. Also ensure label/selector keys appropriately match between resources.


# References

[^1]: k3s quick start: https://docs.k3s.io/quick-start

[^2]: external cluster access: https://docs.k3s.io/cluster-access

[^3]: https://github.com/k3s-io/k3s/discussions/9436

[^4]: API version guide: https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-apiversion-definition-guide.html
