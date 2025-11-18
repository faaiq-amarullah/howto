# Create New MinIO Bucket

## Step 1: Create Bucket

```bash 
mc mb mycluster/rancher-management/etcd-snapshot/management
```

## Step 2: Create Access Policies

Create a read-write policy file `rancher-management-readwrite-policy.json`:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::rancher-management",
        "arn:aws:s3:::rancher-management/*"
      ]
    }
  ]
}
```

Apply the policy:
```bash
mc admin policy create mycluster rancher-management-readwrite rancher-management-readwrite-policy.json
```

## Step 3: Create Users and Assign Policies

```bash
# Create a new user
mc admin user add mycluster rancher-management-user SecurePassword123!

# Assign policy to user
mc admin policy attach mycluster rancher-management-readwrite --user rancher-management-user

# List users
mc admin user list mycluster
```

## Step 4: Set Bucket Policies

```bash
# Set private policy
mc anonymous set private mycluster/rancher-management

# Get bucket policy
mc anonymous get mycluster/rancher-management
```
