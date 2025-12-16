## Install govc client

```bash
export GOVC_INSECURE=1
export GOVC_URL='https://administrator@vsphere.local:<PASSWORD>@vc802.gs.labs'
curl -L -o - "https://github.com/vmware/govmomi/releases/latest/download/govc_$(uname -s)_$(uname -m).tar.gz" | tar -C /usr/local/bin -xvzf - govc
```

## Enable UUID

```bash
govc vm.change -vm /Sydney/vm/control1 -e disk.enableUUID=TRUE
govc vm.change -vm /Sydney/vm/worker1 -e disk.enableUUID=TRUE
```

## Extract VM UUID

```bash
govc vm.info -json -dc=Sydney -vm.ipath="/Sydney/vm/control1" -e=true | jq -r ' .virtualMachines[] | .config.uuid '
```

## Edit /etc/rancher/k3s/config.yaml file

```yaml
kubelet-args:
  # Add this line
  cloud-provider: external
  provider-id: vsphere://<UUID>"
```

## Install CPI Plugin

1. Click **☰** > **Cluster Management**.
2. Go to the cluster where the vSphere CPI plugin will be installed and click Explore.
3. Click **Apps** > **Charts**.
4. Search **vSphere CPI** and select.
5. Fill out the required vCenter details.
6. vSphere CPI initializes all nodes with ProviderID which is needed by the vSphere CSI driver. Check if all nodes are initialized with the ProviderID before installing CSI driver with the following command:

```bash
kubectl describe nodes | grep "ProviderID"
```

## Install CSI 
apply to create namespace
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/vsphere-csi-driver/refs/heads/master/manifests/vanilla/namespace.yaml
```

example secret config csi for vmfs

```bash
vim csi-vsphere.conf

[Global]
cluster-id = "talos-cluster"
cluster-distribution = "Talos"

[VirtualCenter "172.20.0.1"]
insecure-flag = "true"
user = "administrator@vsphere.local"
password = "xxx"
port = "443"
datacenters = "datacenter"
default-datastore = "datastore"
create secret
```

```bash
kubectl create secret generic vsphere-config-secret --from-file=csi-vsphere.conf --namespace=vmware-system-csi
apply csi driver
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/vsphere-csi-driver/refs/heads/master/manifests/vanilla/vsphere-csi-driver.yaml
```

add label vmware-system-csi for privileged escalation

```bash
kubectl label ns vmware-system-csi pod-security.kubernetes.io/enforce=privileged --overwrite
kubectl label ns vmware-system-csi pod-security.kubernetes.io/audit=privileged --overwrite
kubectl label ns vmware-system-csi pod-security.kubernetes.io/warn=privileged --overwrite
```

testing storage policy

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: vmfs-default
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: csi.vsphere.vmware.com
parameters:
  datastoreurl: "ds:///vmfs/volumes/5fdfb4d2-2f0a7f8a/" {optional}
reclaimPolicy: Delete
volumeBindingMode: Immediate
```

jika datastoreurl kosong, CSI akan pilih datastore default dari host ESXi yang terkait node.

datastoreurl can get using govc.
```yaml
govc datastore.info -json <datastore_name> | grep url
```