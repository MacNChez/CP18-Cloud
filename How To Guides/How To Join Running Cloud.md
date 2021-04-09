# This guide will allow you to join the current running cloud as it stands. This is highly subject to change.



## Step 1: Install ZeroTier CLI Tool:
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


## Step 2: Install K3S as a new Master:
In this scenario you will be joining your node as a new master to be joined to the cluster

This scenario has the etcd workload running on an external Azure VM - this is subject to change

### Install K3s:
```
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.18.16+k3s1 K3S_DATASTORE_ENDPOINT='http://<public VM IP>:2379' INSTALL_K3S_EXEC="--flannel-iface ztppi4wr24" sh -
```
-You will need to get public IP before installing
-Wait for installation to finish, verify with 'kubectl get nodes'














# Getting Persistent Docker Storage via GlusterFS
This process will build a Gluster volume inside of USB-attached storage on the Raspberry Pi cluster for access via Docker or K3s
-Adding a new Node to the cluster will require changes to this process

## Baseline Upgrade

```
sudo apt-get update
sudo apt-get upgrade -y
```

## Ensuring all Docker Hosts known in hosts file:

```
sudo nano /etc/hosts

Example: 
192.168.1.67 docker-master
192.168.1.107 docker-node1
192.168.1.117 docker-node2
```

## Deploying Swarm
Install Docker
```
sudo apt-get install docker.io -y

sudo systemctl start docker
sudo systemctl enable docker

sudo usermod -aG docker $USER
sudo newgrp docker

#only done on master(s)
docker swarm init --advertise-addr 'xxx.xxx.xxx.xxx'
#nodes will take command output and run to join swarm

```
## Install xfsprogs and Convert USB storage
This will be used to create the file system on the external storage
```
sudo apt-get install xfsprogs
mkfs.xfs -i size=512 /dev/sda1
```
### Update fstab to ensure USB gets mounted to the specified directory upon startup
```
sudo nano /etc/fstab
/dev/sda1 /mnt/usb0 xfs defaults 1 2
```
-ENSURE both specified directories exist or your device will fail to boot
### Mount the USB
```
mount -a
```
-Can validate by checking the output of 'mount'


## Install GlusterFS
```
sudo apt-get install software-properties-common -y
sudo add-apt-repository ppa:gluster/glusterfs-3.12
sudo apt-get update
sudo apt install glusterfs-server -y

sudo systemctl start glusterd
sudo systemctl enable glusterd
```

## Probe Node to add to Cluster

```
gluster peer probe (docker-node1)
#node name should be same as in host file - check with someone else already in cluster to get hostname/ip to add to file
```
-validate with output of gluster pool list


## Create Directories for Mounting

```
sudo mkdir /gfdata
sudo mkdir /mnt/usb0/data
sudo chmod 777 /gfdata #Gives full permission to the folder to all users/groups on the device - can update as needed
```

## Join running Gluster Volume

```
gluster volume add-brick data replica (depends on how many nodes currently running/number of bricks) localhost:/mnt/usb0/data
gluster vol status
```


### Update fstab to mount Gluster volume on startup to an easily accessed directory
```
sudo echo 'localhost:/data /gfdata glusterfs defaults,_netdev,backupvolfile-server=localhost 0 0' >> /etc/fstab
sudo mount.glusterfs localhost:/data /gfdata
chown -R root:docker /gfdata
```
-If permission denied, run sudo su and attempt the command again without sudo


### Verify mounting:
```
df -h
```

## Any Files/Folders made in /gfdata on a node will persist in /mnt/usb0/data ie; the local storage for the USB drive



