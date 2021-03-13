# How to Create a Highly Available K3S Multi-Master Cluster over the Internet via locally hosted ZeroTier & embedded etcd on Raspberry Pi's

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


## Step 2(A): Install K3S as a new Master:
In this scenario you will be joining your node as a new master to be joined to the cluster

do NOT join as a master if your device will not be able to handle the etcd database

### Add variables to provide latest release, user control over k3s cmd, and set the ZeroTier interface as the external node IP for flannel
```
export K3S_KUBECONFIG_MODE="644"
export INSTALL_K3S_CHANNEL="latest"
export INSTALL_K3S_EXEC="--flannel-iface zt7cqhx4cs" 
```
### Install K3s:
```
curl -sfL https://get.k3s.io |  sh -
```
-Wait for installation to finish, verify with 'kubectl get nodes'
-Once install is successful, halt process to begin join to cluster
```
sudo systemctl stop k3s.service
```

### You will Need to Request the K3s Node Token to Join:
```
INSTALL_K3S_EXEC="--flannel-iface <zerotier-interface>" K3S_TOKEN="<node-token>" k3s server --server https://master1:6443--flannel-iface <zerotier-interface>
```
-Wait a couple of minutes for node to join, and press ctrl-c to halt the process so we can begin running it in the background
```
systemctl restart k3s.service
```
### Verify the Node has joined the cluster as a master:
```
kubectl get nodes
```
