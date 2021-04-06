# How to Create a Highly Available K3S Multi-Master Cluster over the Internet via locally hosted ZeroTier & Azure-hosted etcd on Raspberry Pi's
## Note: This will currently deploy with the default Traefik ingress controller, which is subject to change



## Step One: Install ZeroTier CLI Tool:
```
sudo curl -s https://install.zerotier.com | sudo bash
sudo chown 777 /var/lib/zerotier-one/authtoken.secret #Gives full access to authentication token to allow ZertoTier Network joining
```
- You will need to request the network ID of the ZeroTier private network, and wait for an approved member to accept the request
```
zerotier-cli join <network-id>
```
- You are successfully join when you see a new interface added in the output of 'ifconfig'

### REQUIRED FOR K3s ON RASPBERRY PI ARM ARCHITECTURE:
Enable cgroups in cmdline.txt boot document:
```
sudo nano /boot/cmdline.txt
```
-Append "cgroup_memory=1 cgroup_enable=memory" to the end of the line and restart the machine


## Step 2(A): Install K3S as a new Master:
In this scenario you will be joining your node as a new master to be joined to the cluster

This scenario has the etcd workload running on an external Azure VM - this is subject to change

### Install K3s:
```
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.18.16+k3s1 K3S_DATASTORE_ENDPOINT='http://<public VM IP>:2379' INSTALL_K3S_EXEC="--flannel-iface ztppi4wr24" sh -
```
-You will need to get public IP before installing
-Wait for installation to finish, verify with 'kubectl get nodes'

## Step 2(B): Install K3s as a Worker Node:
This will install only the K3s Agent on the machine, and will not act as a Master for HA purposes
```
curl -sfL https://get.k3s.io | K3S_URL=https://<any cluster master zerotier IP>:6443 K3S_TOKEN=<K3s-node-token> INSTALL_K3S_EXEC="--flannel-iface <zerotier-interface>" sh -
```
### By default any worker node will not have access to manage K3s Cluster, will need to install /etc/rancher/k3s/k3s.yaml file into ~/.kube/config and set master IP


