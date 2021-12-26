# helm-aws-prometheus-stack

## Accessing AlertManager UI

To visit the Alertmanager UI you can run

```
kubectl --namespace platform  port-forward svc/prometheus-alertmanager 9093
```

then visit localhost:9093

## Reload AlertManager config

If the prometheus operator does not reload the config automatically you can manually initiate the config reload by hitting below endpoint after port forwarding:

```
curl -X POST http://127.0.0.1:9093/-/reload
```
