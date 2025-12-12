# Schedule Prometheus + Grafana stack via Nova

Create the namespace policy and namespace `monitoring` on which Prometheus and Grafana will be installed:

```ssh
KUBECONFIG=/Users/selvik/.nova/nova/nova-kubeconfig kubectl apply -f prom-grafana-namespace-policy.yaml
KUBECONFIG=/Users/selvik/.nova/nova/nova-kubeconfig kubectl apply -f prom-grafana-namespace.yaml
```

Create the Nova Scheduling policy to deploy the Prometheus Grafana stack to a specific workload cluster:

```ssh
KUBECONFIG=/Users/selvik/.nova/nova/nova-kubeconfig kubectl apply -f prom-grafana-policy.yaml
```

This steps installs Prometheus, Alertmanager and Grafana via `helm`:

```ssh
KUBECONFIG=/Users/selvik/.nova/nova/nova-kubeconfig helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

```ssh
KUBECONFIG=/Users/selvik/.nova/nova/nova-kubeconfig  helm repo update
```

```ssh
KUBECONFIG=/Users/selvik/.nova/nova/nova-kubeconfig  helm install prom prometheus-community/kube-prometheus-stack \
  --namespace monitoring --create-namespace\
  --set prometheus.prometheusSpec.resources.requests.memory=512Mi \
  --set prometheus.prometheusSpec.resources.requests.cpu=100m \
  --set alertmanager.alertmanagerSpec.resources.requests.memory=128Mi \
  --set alertmanager.alertmanagerSpec.resources.requests.cpu=50m \
  --set grafana.resources.requests.memory=128Mi \
  --set grafana.resources.requests.cpu=50m \
  --wait --timeout 5m
```

We now label the CRDs for Nova scheduling to work: 

```ssh
PROM_CRDS=(
  "alertmanagerconfigs.monitoring.coreos.com"
  "alertmanagers.monitoring.coreos.com" 
  "podmonitors.monitoring.coreos.com"
  "probes.monitoring.coreos.com"
  "prometheusagents.monitoring.coreos.com"
  "prometheuses.monitoring.coreos.com"
  "prometheusrules.monitoring.coreos.com"
  "scrapeconfigs.monitoring.coreos.com"
  "servicemonitors.monitoring.coreos.com"
  "thanosrulers.monitoring.coreos.com"
)
```

```ssh
# Patch with idempotent merge (safe to re-run)
for CRD in "${PROM_CRDS[@]}"; do
  echo "Labeling $CRD..."
  KUBECONFIG=/Users/selvik/.nova/nova/nova-kubeconfig kubectl patch crd "$CRD" --type=merge -p '{
    "metadata": {
      "labels": {
        "app.kubernetes.io/instance": "prom"
      }
    }
  }'
done
```

We now check that the pods are running on the workload cluster. 
Please replace the workload cluster name `selvik-12232` with the name of your workload cluster:

```ssh
% kubectl get pods -n monitoring  --context selvik-12232
NAME                                                   READY   STATUS    RESTARTS   AGE
prom-grafana-7c5f9cd65b-c4jgl                          3/3     Running   0          7m14s
prom-kube-prometheus-stack-operator-6645565fff-l8s24   1/1     Running   0          7m15s
prom-kube-state-metrics-5fb556dc75-d8mjv               1/1     Running   0          7m15s
prom-prometheus-node-exporter-7gt4c                    1/1     Running   0          7m15s
prom-prometheus-node-exporter-kc9c4                    1/1     Running   0          7m15s
```

The Grafana dashboard can be accessed as follows:

```ssh
kubectl port-forward -n monitoring svc/prom-grafana 3000:80 --context selvik-12232
```

