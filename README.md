# kube-prometheus

Components included in this package:

* The [Prometheus Operator](https://github.com/prometheus-operator/prometheus-operator)
* Highly available [Prometheus](https://prometheus.io/)
* Highly available [Alertmanager](https://github.com/prometheus/alertmanager)
* [Prometheus node-exporter](https://github.com/prometheus/node_exporter)
* [Prometheus blackbox-exporter](https://github.com/prometheus/blackbox_exporter)
* [Prometheus Adapter for Kubernetes Metrics APIs](https://github.com/kubernetes-sigs/prometheus-adapter)
* [kube-state-metrics](https://github.com/kubernetes/kube-state-metrics)
* [Grafana](https://grafana.com/)
* [Loki](https://grafana.com/oss/loki/)
* [Promtail](https://grafana.com/docs/loki/latest/send-data/promtail/installation/)

This stack is meant for cluster monitoring with logs explorere, so it is pre-configured to collect metrics and logs from all Kubernetes components. In addition to that it delivers a default set of dashboards and alerting rules. 

## Prerequisites

You will need a Kubernetes cluster, that's it! By default it is assumed, that the kubelet uses token authentication and authorization, as otherwise Prometheus needs a client certificate, which gives it full access to the kubelet, rather than just the metrics. Token authentication and authorization allows more fine grained and easier access control.

This means the kubelet configuration must contain these flags:

* `--authentication-token-webhook=true` This flag enables, that a `ServiceAccount` token can be used to authenticate against the kubelet(s). This can also be enabled by setting the kubelet configuration value `authentication.webhook.enabled` to `true`.
* `--authorization-mode=Webhook` This flag enables, that the kubelet will perform an RBAC request with the API to determine, whether the requesting entity (Prometheus in this case) is allowed to access a resource, in specific for this project the `/metrics` endpoint. This can also be enabled by setting the kubelet configuration value `authorization.mode` to `Webhook`.

This stack provides [resource metrics](https://github.com/kubernetes/metrics#resource-metrics-api) by deploying
the [Prometheus Adapter](https://github.com/kubernetes-sigs/prometheus-adapter).
This adapter is an Extension API Server and Kubernetes needs to be have this feature enabled, otherwise the adapter has
no effect, but is still deployed.

## Compatibility

The following Kubernetes versions are supported and work as we test against these versions in their respective branches. But note that other versions might work!

| kube-prometheus stack                                                      | Kubernetes 1.27 | Kubernetes 1.28 |
| -------------------------------------------------------------------------- | --------------- | --------------- |
| [`main`](https://github.com/prometheus-operator/kube-prometheus/tree/main) | ✔               | ✔               |

## Quickstart

```shell
# Create the namespace and CRDs, and then wait for them to be available before creating the remaining resources
# Note that due to some CRD size we are using kubectl server-side apply feature which is generally available since kubernetes 1.22.
# If you are using previous kubernetes versions this feature may not be available and you would need to use kubectl create instead.
kubectl apply --server-side -f manifests/custom-resource-defination-setup

#Check for CRD installed properly and met with our requirements
kubectl wait --for condition=Established --all CustomResourceDefinition --namespace=monitoring

# For Infrastructure kubernetes monitoring 
kubectl apply -f manifests/infrastructure-monitoring-setup/

# For kubernetes pods containers logs monitoring
kubectl apply -f manifests/logs-monitoring-setup/
```

```shell
# To delete the complete stack from server
kubectl delete --ignore-not-found=true -f manifests/logs-monitoring-setup/ manifests/infrastructure-monitoring-setup/ -f manifests/custom-resource-defination-setup
```

```shell
# To check and validate the components deployed in stack
kubectl get crd -A
kubectl get ns | grep "monitoring"
kubectl get all -n monitoring
kubectl get pod -n monitoring
kubectl get svc -n monitoring
kubectl get secrets -n monitoring
kubectl get cm -n monitoring
```

##### To check all available additional dashboards for grafana use below directory

```
# for loki logs explorer dashboards please import the `grafana-k8s-loki.json` file 
grafana/dashboards/grafana-k8s-loki.json

# for other infrastructure dashboards except default dashboards please imports from here
grafana/dashboards/grafana-blackbox.json
grafana/dashboards/grafana-linux-node-exporter.json
grafana/dashboards/grafan-node-exporter.json
```


