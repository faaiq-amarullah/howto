## Install helm chart

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack --version 80.4.1 \
     --namespace monitoring --create-namespace \
     --set "alertmanager.enabled=false" --set "grafana.enabled=false" \
     --set "prometheus.service.type=NodePort" --set "prometheus.prometheusSpec.retention=30d" \
     --set "prometheus.prometheusSpec.retentionSize=50GiB"
```
