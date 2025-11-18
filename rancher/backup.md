# Backup Rancher
The backup-restore operator needs to be installed in the local cluster, and only backs up the Rancher app. The backup and restore operations are performed only in the local Kubernetes cluster.

Note that the rancher-backup operator version 2.x.x is for Rancher v2.6.x.

# Prerequisites

The Rancher version must be v2.5.0 and up.
Refer here for help on restoring an existing backup file into a v1.22 cluster in Rancher v2.6.3.

## Step 1: Create Secret

```bash
kubectl create secret generic s3-creds \
  --from-literal=accessKey=<ACCESS KEY> \
  --from-literal=secretKey=<SECRET KEY>
```

## Step 2: Install the Rancher Backup operator

The backup storage location is an operator-level setting, so it needs to be configured when the Rancher Backups application is installed or upgraded.

Backups are created as .tar.gz files. These files can be pushed to S3 or Minio, or they can be stored in a persistent volume.

1. In the upper left corner, click ☰ > C`Cluster Management`.
2. On the Clusters page, go to the local cluster and click `Explore`. The local cluster runs the Rancher server.
3. Click `Apps` > `Charts`.
4. Click `Rancher Backups`.
5. Click `Install`.
6. Configure the default storage location. For help, refer to the storage configuration section.
7. Click `Install`.

## Step 3: Perform a Backup

To perform a backup, a custom resource of type Backup must be created.

1. In the upper left corner, click ☰ > Cluster Management.
2. On the Clusters page, go to the local cluster and click Explore.
3. In the left navigation bar, click Rancher Backups > Backups.
4. Click Create.
5. Create the Backup with the form, or with the YAML editor.
6. For configuring the Backup details using the form, click Create and refer to the configuration reference and to the examples.
7. For using the YAML editor, we can click Create > Create from YAML. Enter the Backup YAML. This example Backup custom resource would create encrypted recurring backups in S3. The app uses the credentialSecretNamespace value to determine where to look for the S3 backup secret:

```yaml
apiVersion: resources.cattle.io/v1
kind: Backup
metadata:
  name: s3-recurring-backup
spec:
  storageLocation:
    s3:
      credentialSecretName: s3-creds
      credentialSecretNamespace: default
      bucketName: <BUCKET NAME>
      folder: <FOLDER NAME>
      endpoint: <ENDPOINT>
  resourceSetName: rancher-resource-set-full
  schedule: "@every 1h"
  retentionCount: 10
```

8. Click Create.

## Reference
Visit https://ranchermanager.docs.rancher.com/how-to-guides/new-user-guides/backup-restore-and-disaster-recovery/back-up-rancher
