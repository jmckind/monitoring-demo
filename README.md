# OpenShift Monitoring Demo

This demo shows the process of using OpenShift Monitoring to monitor user-defined projects based on the official [documentation][link_docs].

## Sample Application

The [sample application][link_sample] is a simple web server that exposes basic Prometheus metrics.

Create a new namespace for the application.

```
kubectl create namespace myapp
```

Deploy the sample application using the manifests.

```shell
kubectl apply -n myapp -f prometheus-example-app.yaml
```

Save the external hostname for the application.

```shell
export ROUTE_HOST=$(kubectl get route -n myapp prometheus-example-app -o jsonpath='{.spec.host}')
```

Verify that the sample application is accessible externally. Note that a `404 Not Found` error is expected for the second command.

```shell
curl -ik https://${ROUTE_HOST}/
curl -ik https://${ROUTE_HOST}/err
```

This simple web application uses Prometheus counters to track the number of requests for each endpoint.

## Configure OpenShift

Create the `cluster-monitoring-config` ConfigMap in the `openshift-monitoring` namespace and enable user workload monitoring.

```shell
kubectl apply -f cluster-monitoring-config.yaml
```

Grant user-level access to manage monitoring resources such as the `ServiceMonitor` and`PrometheusRule` resources.

Privileges are granted by assigning one of the following monitoring roles:

* **monitoring-rules-view**: READ access to PrometheusRule custom resources
* **monitoring-rules-edit**: (CRUD access to PrometheusRule custom resources)
* **monitoring-edit**: (same as monitoring-rules-edit but adds CRUD for ServiceMonitor/PodMonitor resources)

Delegation is also possible using the following role in the `openshift-user-workload-monitoring` namespace.

* user-workload-monitoring-config-edit

```shell
kubectl apply -n myapp -f monitoring-role-binding.yaml
```

## Metrics Collection

Create a ServiceMonitor to scrape the application metrics.

```shell
kubectl apply -n myapp -f service-monitor.yaml
```

After a few moments, the metrics should be available in the Monitoring dashboard.

Select the `Developer` user, select the `Monitoring` section on the left and then click the `Metrics` tab.

Enter a custom query and the following metrics should be available.

* http_requests_total
* http_request_duration_seconds_bucket
* http_request_duration_seconds_sum
* http_request_duration_seconds_count
* version

[link_sample]:https://github.com/brancz/prometheus-example-app
[link_docs]:https://docs.openshift.com/container-platform/4.6/monitoring/enabling-monitoring-for-user-defined-projects.html
