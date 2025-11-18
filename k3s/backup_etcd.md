# Configure K3s ETCD Snapshot

## Step 1: Create secret

Create `rancher-management-secret.yaml` file:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: k3s-etcd-snapshot-s3-config
  namespace: kube-system
type: etcd.k3s.cattle.io/s3-config-secret
stringData:
  etcd-s3-endpoint: "<ENDPOINT URL>"
  etcd-s3-access-key: "<ACCESS KEY>"
  etcd-s3-secret-key: "<SECRET KEY>"
  etcd-s3-bucket: "<BUCKET NAME>"
  etcd-s3-folder: "<FOLDER NAME>"
  etcd-s3-skip-ssl-verify: "false"
  etcd-s3-insecure: "false"
```

Apply secret:
```bash
kubectl apply -f rancher-management-secret.yaml
```

## Step 2: Edit config.yaml File on K3s Server Node

```yaml
# Add this line
etcd-s3: "true"
etcd-s3-config-secret: "k3s-etcd-snapshot-s3-config"
etcd-snapshot-name: "etcd-snapshot"
etcd-snapshot-schedule-cron: "0 */5 * * *"
etcd-snapshot-retention: "10"
```

## Step 3: Restart K3s service

```bash
sudo systemctl restart k3s
sudo systemctl status k3s
```
