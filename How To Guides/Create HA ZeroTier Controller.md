# How to Create a Highly Available ZeroTier Controller hosted on a Highly Available K3s Cluster
## Note: This will only be done ONCE at the start of the cluster lifecycle, and can be scaled up and down as needed, but creating a new controller will result in a new network ID



## Step One: Create YAML file for HELM deployment:
```
sudo nano ztncui.yaml


apiVersion: helm.cattle.io/v1
kind: HelmChart
metadata:
  name: zerotier-controller-ui
  namespace: default
spec:
  helmVersion: v1
  name: ztncui
  chart: https://iotops.gitlab.io/charts/ztncui-0.1.0.tgz
  targetNamespace: default
```
- This will deploy a chart from the specified repository, which will automate the deployment of the resource required for the zerotier controller and UI


## Step 2: Apply the .YAML file
```
kubectl apply -f ztncui.yaml
```
-Watch delployment progress: kubectl get pods --all-namespaces --watch

## Step 3: Port Forward the Service to access the GUI
By default, the service will be running but will not be accessable outside of the pods running it, and will require forwarding of the port to manage access
```
kubectl port-forward svc/zerotier-controller-ui-ztncui 6443:6443
```
-Default login is admin:password



