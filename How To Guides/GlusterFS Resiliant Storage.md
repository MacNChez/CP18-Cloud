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
gluster peer probe (docker-node1); gluster peer probe (docker-node2);
#node names should be same as in host file
```
-validate with output of gluster pool list


## Create Directories for Mounting

```
sudo mkdir /gfdata
sudo mkdir /mnt/usb0/data
sudo chmod 777 /gfdata #Gives full permission to the folder to all users/groups on the device - can update as needed
```

## Create Gluster Volume and mount to USB storage
```
sudo gluster volume create data replica 2 transport tcp master1:/mnt/usb0/data master4:/mnt/usb0/data
sudo gluster volume start data 
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



