# Scheduling the Prometheus + Grafana stack via Nova


## Namespace Creation

Create the namespace policy and namespace `monitoring` on which Prometheus and Grafana will be installed:

```ssh
KUBECONFIG=/Users/selvik/.nova/nova/nova-kubeconfig kubectl apply -f prom-grafana-namespace-policy.yaml
KUBECONFIG=/Users/selvik/.nova/nova/nova-kubeconfig kubectl apply -f prom-grafana-namespace.yaml
```

## Helm chart Installation

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

Successful install output:

```
NAME: prom
LAST DEPLOYED: Fri Dec 12 05:48:05 2025
NAMESPACE: monitoring
STATUS: deployed
REVISION: 1
NOTES:
kube-prometheus-stack has been installed. Check its status by running:
  kubectl --namespace monitoring get pods -l "release=prom"

Get Grafana 'admin' user password by running:

  kubectl --namespace monitoring get secrets prom-grafana -o jsonpath="{.data.admin-password}" | base64 -d ; echo

Access Grafana local instance:

  export POD_NAME=$(kubectl --namespace monitoring get pod -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=prom" -oname)
  kubectl --namespace monitoring port-forward $POD_NAME 3000

Get your grafana admin user password by running:

  kubectl get secret --namespace monitoring -l app.kubernetes.io/component=admin-secret -o jsonpath="{.items[0].data.admin-password}" | base64 --decode ; echo


Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator.
```


### Labeling CRDs

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

## Verify Installation

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

