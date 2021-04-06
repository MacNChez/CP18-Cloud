# Getting Persistent Docker Storage via GlusterFS

(Currently configured for Ubuntu18.04 - will have to find differences for Pi-based architectures)

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

## Install GlusterFS
```
sudo apt-get install software-properties-common -y
sudo add-apt-repository ppa:gluster/glusterfs-3.12
sudo apt-get update
sudo apt install glusterfs-server -y

sudo systemctl start glusterd
sudo systemctl enable glusterd
```

## Generate SSH Keys
```
ssh-keygen -t rsa
```

## Probing Nodes

```
sudo -s
gluster peer probe (docker-node1); gluster peer probe (docker-node2);
#node names should be same as in host file
gluster pool list
```

## Create Gluster Volume

```
sudo mkdir -p /gluster/volume1

sudo gluster volume create staging-gfs replica 3 docker-master:/gluster/volume1 docker-node1:/gluster/volume1 docker-node2:/gluster/volume1 force
#This would create 3 replicas across the two nodes specified - will change depending on deployment

sudo gluster volume start staging-gfs
```

### Ensure volume mounts on reboot - mount to /mnt shared directory
```
sudo -s
echo 'localhost:/staging-gfs /mnt glusterfs defaults,_netdev,backupvolfile-server=localhost 0 0' >> /etc/fstab
mount.glusterfs localhost:/staging-gfs /mnt
chown -R root:docker /mnt
exit
```

### Verify mounting:
```
df -h
```

## Any Files/Folders made in /mnt on a node will persist in /gluster/volume1



