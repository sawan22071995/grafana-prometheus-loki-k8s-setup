# 🚀 Grafana-Prometheus-Promtail-Loki-Kubernetes 🚀

## ✅Introduction

#### 📌Grafana

- Grafana is an open-source data visualization and monitoring tool that provides a unified interface for querying, visualizing, and analyzing metrics from various data sources. 

- It offers a rich set of dashboards, graphs, and alerting capabilities.

#### 📌Prometheus

- Prometheus is an open-source monitoring and alerting system that collects and stores time-series data from various sources, including applications, databases, and systems. 

- It is designed to monitor modern, dynamic environments and provides a powerful query language for retrieving and analyzing data.

#### 📌Loki

- Loki is a horizontally scalable, highly available log aggregation system inspired by Prometheus. 

- It is designed to handle large volumes of logs and provides efficient log querying and analysis capabilities.

#### 📌Promtail

- Promtail is an agent that ships logs from various sources (e.g., files, Docker containers, Kubernetes pods) to Loki. 

- It facilitates log collection and provides features like multiline support, label management, and filtering.

#### 📌AlertManager

- AlertManager is a component of the Prometheus monitoring system that handles alerts generated by Prometheus rules. 

- It deduplicates, groups, and routes alerts to various notification channels (e.g., email, Slack, PagerDuty) based on configurable routing rules.

### ✅Components included in this package:

* [Grafana](https://grafana.com/)
* Highly available [Prometheus](https://prometheus.io/)
* The [Prometheus Operator](https://github.com/prometheus-operator/prometheus-operator)
* [Loki](https://grafana.com/oss/loki/)
* [Promtail](https://grafana.com/docs/loki/latest/send-data/promtail/installation/)
* [kube-state-metrics](https://github.com/kubernetes/kube-state-metrics)
* Highly available [Alertmanager](https://github.com/prometheus/alertmanager)
* [Prometheus Adapter for Kubernetes Metrics APIs](https://github.com/kubernetes-sigs/prometheus-adapter)

This stack is meant for cluster monitoring with logs explorere, so it is pre-configured to collect metrics and logs from all Kubernetes components. In addition to that it delivers a default set of dashboards and alerting rules. 

## ✅Prerequisites

You will need a Kubernetes cluster, that's it! By default it is assumed, that the kubelet uses token authentication and authorization, as otherwise Prometheus needs a client certificate, which gives it full access to the kubelet, rather than just the metrics. Token authentication and authorization allows more fine grained and easier access control.

This means the kubelet configuration must contain these flags:

* `--authentication-token-webhook=true` This flag enables, that a `ServiceAccount` token can be used to authenticate against the kubelet(s). This can also be enabled by setting the kubelet configuration value `authentication.webhook.enabled` to `true`.
* `--authorization-mode=Webhook` This flag enables, that the kubelet will perform an RBAC request with the API to determine, whether the requesting entity (Prometheus in this case) is allowed to access a resource, in specific for this project the `/metrics` endpoint. This can also be enabled by setting the kubelet configuration value `authorization.mode` to `Webhook`.

This stack provides [resource metrics](https://github.com/kubernetes/metrics#resource-metrics-api) by deploying
the [Prometheus Adapter](https://github.com/kubernetes-sigs/prometheus-adapter).
This adapter is an Extension API Server and Kubernetes needs to be have this feature enabled, otherwise the adapter has
no effect, but is still deployed.

## ✅Compatibility

The following Kubernetes versions are supported and work as we test against these versions in their respective branches. But note that other versions might work!

| kube-prometheus stack                                                      | Kubernetes 1.27 | Kubernetes 1.28 |
| -------------------------------------------------------------------------- | --------------- | --------------- |
| [`main`](https://github.com/prometheus-operator/kube-prometheus/tree/main) | ✔               | ✔               |

## ✅Quickstart

```shell
# Create the namespace and CRDs, and then wait for them to be available before creating the remaining resources
# Note that due to some CRD size we are using kubectl server-side apply feature which is generally available since kubernetes 1.22.
# If you are using previous kubernetes versions this feature may not be available and you would need to use kubectl create instead.
kubectl apply --server-side -f manifests/custom-resource-defination-setup

#Check for CRD installed properly and met with our requirements
kubectl wait --for condition=Established --all CustomResourceDefinition --namespace=monitoring

# For Infrastructure kubernetes monitoring 
kubectl apply -f manifests/k8s-workload-monitoring-setup/

# For kubernetes pods containers logs monitoring
kubectl apply -f manifests/logs-monitoring-setup/
```

```shell
# To delete the complete stack from server
kubectl delete --ignore-not-found=true -f manifests/logs-monitoring-setup/ manifests/k8s-workload-monitoring-setup/ -f manifests/custom-resource-defination-setup
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

## ✅ Grafana Dashboards

##### To check all available additional dashboards for grafana use below directory

```
# for loki logs explorer dashboards please import the `grafana-k8s-loki.json` file 
grafana/dashboards/grafana-k8s-loki.json

```

## ✅ `Kubetail` - a Live log Viewer Web Interface for Kubernestes
Kubetail providing an easy-to-use, web-based interface that allows you to view all the logs for a set of Kubernetes workloads (e.g. Deployment, CronJob, StatefulSet) simultaneously, in real-time. Under the hood, it uses your cluster's Kubernetes API to monitor your workloads and detect when a new workload container gets created or an old one deleted.

- **Key features**
    - Small and resource efficient (<30MB of memory, negligible CPU)
    - View log messages in real-time
    - View logs that are part of a specific workload (e.g. Deployment, CronJob, StatefulSet)
    - Detects creation and deletion of workload containers and adds their logs to the viewing stream automatically
    - Uses your Kubernetes API so log messages never leave your possession (private by default)
    - Filter logs based on time
    - Filter logs based on node properties such as availability zone, CPU architecture or node ID
    - Color-coded log lines to distinguish between different containers
    - A clean, easy-to-use interface

- **Install**

```
kubectl create ns kubetail
kubectl apply -f manifests/kubetail/kubetail-clusterauth.yaml

```

- **Access on Browser**

```
https://<loadBalancer_ip>

```

- **Demo**

```
https://www.kubetail.com/demo

```

- **Reference**
https://github.com/kubetail-org/kubetail

