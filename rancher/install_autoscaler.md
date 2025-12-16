# Enabling Cluster Autoscaling on Rancher-managed RKE2 Cluster

## Pre-requisites

Using Rancher 2.7+, create a RKE2 cluster using vsphere (or public cloud) node driver.

## Step 1 - Create a secret to connect to the Rancher server which manages this RKE2 cluster.

1. Click on the top right avatar in Rancher UI to show the pull down menu and choose `Accounts and API Keys`.
2. Create a new API Key and capture its token generated for this new API account
3. Edit the manifest below with your own Rancher URL and API token. Apply this onto the `kube-system` namespace of this RKE2 cluster.

For example,
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: cluster-autoscaler-cloud-config
  namespace: kube-system
type: Opaque
stringData:
  cloud-config: |-
    url: https://rancher.k3s.my.test
    token: token-r.........rngnqrnt84fm
    clusterName: cluster-autoscale-demo
    clusterNamespace: fleet-default
```

## Step 2 - Deploy cluster autoscaler on the RKE2 cluster

After the `cluster-autoscale-demo` RKE2 cluster provisioning completed, switch to Cluster Explorer. 
Navigate to menu `Apps` > `Repositories`. Add a new repo:

1. name: `autoscaler`
2. repo: `https://kubernetes.github.io/autoscaler`

After the repo is added, deploy the autoscaler helm chart on the RKE2 cluster

1. namespace: `kube-system`
2. project : `System`
3. helm chart values override as below.

```yaml
autoDiscovery:
  clusterName: cluster-autoscale-demo

cloudConfigPath: /config/cloud-config
cloudProvider: rancher

extraArgs:
  scale-down-utilization-threshold: 0.5
  scale-down-unneeded-time: 1m

extraVolumeSecrets:
  cluster-autoscaler-cloud-config:
    mountPath: /config
    name: cluster-autoscaler-cloud-config
```

Or you might be use this command to deploy cluster autoscaler with CLI.

```bash
# Add cluster autoscaler repo
helm repo add autoscaler https://kubernetes.github.io/autoscaler
helm repo update

# Deploy with additional options
helm install autoscaler autoscaler/cluster-autoscaler \
    --namespace kube-system \
    --set 'autoDiscovery.clusterName'=cluster-autoscale-demo \
    --set 'cloudConfigPath'=/config/cloud-config \
    --set 'cloudProvider'=rancher \
    --set 'extraArgs.scale-down-utilization-threshold'=0.5 \
    --set 'extraArgs.scale-down-unneeded-time'=1m \
    --set 'extraVolumeSecrets.cluster-autoscaler-cloud-config.name'=cluster-autoscaler-cloud-config \
    --set 'extraVolumeSecrets.cluster-autoscaler-cloud-config.mountPath'=/config
```

## Step 3 - Annotate the cluster-autoscale-demo RKE2 cluster

Now, let's annotate the cluster machine pool autoscaling settings on this RKE2 cluster. 
Go to `Cluster Management`, locate the `cluster-autoscale-demo` cluster and edit the YAML settings.

Add the following annotation under `worker` machine pool.

```yaml
machineDeploymentAnnotations:
  cluster.provisioning.cattle.io/autoscaler-max-size: '3'
  cluster.provisioning.cattle.io/autoscaler-min-size: '1'
```

These annotations should be positioned like this part of the YAML.
```yaml
    machinePools:
      - controlPlaneRole: true
        dynamicSchemaSpec: >-
          ....(removed for sanity)
        etcdRole: true
        machineConfigRef:
          kind: VmwarevsphereConfig
          name: nc-cluster-autoscale-demo-master-ccv47
        machineOS: linux
        name: master
        quantity: 1
        unhealthyNodeTimeout: 0s
      - dynamicSchemaSpec: >-
          ....(removed for sanity)
        machineConfigRef:
          kind: VmwarevsphereConfig
          name: nc-cluster-autoscale-demo-worker-9pzc9
        machineDeploymentAnnotations:
          cluster.provisioning.cattle.io/autoscaler-max-size: '3'
          cluster.provisioning.cattle.io/autoscaler-min-size: '1'
        machineOS: linux
        name: worker
        quantity: 1
        unhealthyNodeTimeout: 0s
        workerRole: true
```

## Step 4 - Verify if the autoscaler is working.

Verify if autoscaler works (request 1x2 cpu on a 1 worker node cluster of 2vcpu).

Initially, only 1 of the pods are deployed successfully, leaving the other pod pending due to insufficient cpu error. 
The autoscaler will detect this pending pod and notify Rancher to provision an extra worker node to accommodate the pending pod.

Once the deployment is deleted, the autoscaler will scale down the cluster by removing the unused worker node, after 5 mins of cooling time.

Apply a deployment to `default` namespace with manifest below. 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sleep
  labels:
    app: sleep
spec:
  replicas: 2
  selector:
    matchLabels:
      app: sleep
  template:
    metadata:
      labels:
        app: sleep
    spec:
      containers:
        - image: wardsco/sleep
          name: sleep
          imagePullPolicy: IfNotPresent
          resources:
            requests:
              cpu: 1
```

## Reference

Visit [rancher-rke2-autoscaler-tutorial](https://github.com/dsohk/rancher-rke2-autoscaler-tutorial)
