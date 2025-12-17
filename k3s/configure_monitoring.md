## Install the Prometheus Federator Application

1. Click ☰ > Cluster Management.
2. Go to the cluster that you want to install Prometheus Federator and click Explore.
3. Click Apps -> Charts.
4. Click the Prometheus Federator chart.
5. Click Install.
6. On the Metadata page, click Next.
7. In the Namespaces > Project Release Namespace Project ID field, the System Project is used as the default but can be overridden with another project with similarly limited access. Project IDs can be found with the following command run in the local upstream cluster:

```bash
kubectl get projects -A -o custom-columns="NAMESPACE":.metadata.namespace,"ID":.metadata.name,"NAME":.spec.displayName
```

8. Click Install.

**Result**: The Prometheus Federator app is deployed in the cattle-monitoring-system namespace.

## Enabling the Rancher Performance Dashboard
To enable the Rancher Performance Dashboard:

1. Click ☰ > Cluster Management.
2. Go to the row of the local cluster and click Explore.
3. Click Workloads > Deployments.
4. Use the dropdown menu at the top to filter for All Namespaces.
5. Under the cattle-system namespace, go to the rancher row and click ⋮ > Edit Config
6. Under Environment Variables, click Add Variable.
7. For Type, select Key/Value Pair.
8. For Variable Name, enter CATTLE_PROMETHEUS_METRICS.
9. For Value, enter true.
10. Click Save to apply the change.

## Accessing the Rancher Performance Dashboard
1. Click ☰ > Cluster Management.
2. Go to the row of the local cluster and click Explore.
3. Click Monitoring
4. Select the Grafana dashboard.
5. From the sidebar, click Search dashboards.
6. Enter Rancher Performance Debugging and select it.