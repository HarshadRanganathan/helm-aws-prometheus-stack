# helm-aws-prometheus-stack

Helm chart for setting up:
- The Prometheus Operator
- Highly available Prometheus
- Highly available Alertmanager
- Prometheus node-exporter
- kube-state-metrics

Chart Reference - https://github.com/prometheus-operator/kube-prometheus

## Install/Upgrade Chart

Run below commands to set up prometheus stack:

```
# create config secret
kubectl -n platform create secret generic thanos-objstore-config --from-file=thanos.yaml=stages/prod/thanos-config.yaml

# helm install/upgrade
helm upgrade -i prometheus . -n platform --values=stages/shared-values.yaml --values=stages/prod/prod-values.yaml
```

## Accessing AlertManager UI

To visit the Alertmanager UI you can run

```
kubectl --namespace platform  port-forward svc/prometheus-alertmanager 9093
```

then visit localhost:9093

## Reload AlertManager Config

If the prometheus operator does not reload the config automatically you can manually initiate the config reload by hitting below endpoint after port forwarding:

```
curl -X POST http://127.0.0.1:9093/-/reload
```
