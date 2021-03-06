# How to Configure ZeroTier

## Using Web Based Controller
```
curl -s https://install.zerotier.com | sudo bash

sudo chown $USER:group .zeroTierOneAuthToken
sudo cat /var/lib/zerotier-one/authtoken.secret >>.zeroTierOneAuthToken
chmod 0600 .zeroTierOneAuthToken

zerotier-cli join <network-id>
 ```
## Using Locally Designed Controller

### CREATING CONTROLLER ON PI via K3S(lightweight kubernentes):

### PREREQS TO INSTALLING K3S
- Update cmdline boot doc to enable cgroup memory (requirement for k3s functionality on pi)
```
sudo nano /boot/cmdline.txt
console=serial0,115200 console=tty1 root=PARTUUID=58b06195-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait cgroup_memory=1 cgroup_enable=memory
```
- k3s needs to specify iptables for pi legacy
```
sudo iptables -F
sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
sudo reboot
```

### install k3s:
```
sudo curl -sfL https://get.k3s.io | sh -
sudo nano ztncui.yaml (in user dir)
```
```
apiVersion: helm.cattle.io/v1
kind: HelmChart
metadata:
  name: zerotier-controller-ui
  namespace: kube-system
spec:
  helmVersion: v1
  name: ztncui
  chart: https://iotops.gitlab.io/charts/ztncui-0.1.0.tgz
  targetNamespace: default
```
```
sudo k3s kubectl apply -f ztncui.yaml
```
- validate:
```
sudo k3s kubectl get pods --all-namespaces --watch
```

### start port forward to access gui:
```
sudo kubectl port-forward svc/zerotier-controller-ui-ztncui 3000:3000
```
- default login: admin/password
